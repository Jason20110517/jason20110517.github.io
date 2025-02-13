---
layout: post
title: "Blowing up unordered_map, and how to stop getting hacked on it"
date:   2018-2-19
tags: [OI,C++]
comments: true
author: jason20110517
---

C++ has always had the convenient data structures `std::set` and `std::map`, which are tree data structures whose operations take O(log n) time. With C++11, we finally received a hash set and hash map in `std::unordered_set` and `std::unordered_map`. Unfortunately, I've seen a lot of people on Codeforces get hacked or fail system tests when using these. In this post I'll explain how it's possible to break these data structures and what you can do in order to continue using your favorite hash maps without worrying about being hacked .

So how are they hackable? We always assume hash maps are *O*(1) per operation (insert, erase, access, etc.). But this depends on a key assumption, which is that each item *only runs into \*O*(1) collisions on average*. If our input data is completely random, this is a reasonable assumption. But this is no longer a safe bet when the input isn't random, especially so if someone is adversarially designing inputs to our code (a.k.a. hacking phase). In particular, if they know our hash function, they can easily generate a large number of different inputs that all collide, thus causing an *O*(*n²*) blow-up.

We'll prove that now by blowing up `unordered_map`. In order to do that, we first have to determine exactly how it's implemented. For this we can dig into gcc's implementation on GitHub: https://github.com/gcc-mirror/gcc.

After some searching around we run into [unordered_map.h](https://github.com/gcc-mirror/gcc/blob/5bea0e90e58d971cf3e67f784a116d81a20b927a/libstdc%2B%2B-v3/include/bits/unordered_map.h). Inside the file we can quickly see that `unordered_map` makes use of `__detail::_Mod_range_hashing` and `__detail::_Prime_rehash_policy`. From this we can guess that the map first hashes the input value and then mods by a prime number, and the result is used as the appropriate position in the hash table.

Some further searching for `_Prime_rehash_policy` leads us to [hashtable_c++0x.cc](https://github.com/gcc-mirror/gcc/blob/5bea0e90e58d971cf3e67f784a116d81a20b927a/libstdc%2B%2B-v3/src/c%2B%2B11/hashtable_c%2B%2B0x.cc). Here we can see that there is an array called `__prime_list`, and the hash table has a policy to resize itself when it gets too large. So we just need to find this list of primes.

The one include on this file leads us to [hashtable-aux.cc](https://github.com/gcc-mirror/gcc/blob/5bea0e90e58d971cf3e67f784a116d81a20b927a/libstdc%2B%2B-v3/src/shared/hashtable-aux.cc). Aha, here is the list we're looking for.

One more thing: we need to know the hash function `unordered_map` uses before modding by these primes. It turns out to be quite simple: the map uses `std::hash`, which for integers is simply the identity function. Armed with this knowledge, we can insert lots of multiples of one of these primes to the map in order to get *n²* blow-up. Not all of the primes work though, due to the resizing policy of the map; in order for a prime to work, we need the map to actually resize to this prime at some point in its set of operations. It turns out the right prime depends on the compiler version: for gcc 6 or earlier, 126271 does the job, and for gcc 7 or later, 107897 will work. Run the code below in [Custom Invocation](https://codeforces.com/contest/1033/customtest) and see what output you get.

```cpp
#include <ctime>
#include <iostream>
#include <unordered_map>
using namespace std;

const int N = 2e5;

void insert_numbers(long long x) {
    clock_t begin = clock();
    unordered_map<long long, int> numbers;

    for (int i = 1; i <= N; i++)
        numbers[i * x] = i;

    long long sum = 0;

    for (auto &entry : numbers)
        sum += (entry.first / x) * entry.second;

    printf("x = %lld: %.3lf seconds, sum = %lld\n", x, (double) (clock() - begin) / CLOCKS_PER_SEC, sum);
}

int main() {
    insert_numbers(107897);
    insert_numbers(126271);
}
```

Depending on which compiler version you are using, one of these two numbers will take much longer than the other. There are several other primes that also work; try some more for yourself!

------

Note that for other hash tables like `cc_hash_table` or `gp_hash_table` (see [Chilli](https://codeforces.com/profile/Chilli)'s [helpful post](https://codeforces.com/blog/entry/60737)), it's even easier to hack them. These hash tables use a modulo power of two policy, so in order to make a lot of collisions occur we can simply insert a lot of numbers that are equivalent, say, modulo 2¹⁶.

### Don't want to be hacked?

Let's look at how to safeguard these hash maps from collision attacks. To do this we can write our own custom hash function which we give to the `unordered_map` (or `gp_hash_table`, etc.). The standard hash function looks something like this:

```cpp
struct custom_hash {
    size_t operator()(uint64_t x) const {
        return x;
    }
};
```

However as we mentioned, any predictable / deterministic hash function can be reverse-engineered to produce a large number of collisions, so the first thing we should do is add some non-determinism (via high-precision clock) to make it more difficult to hack:

```cpp
struct custom_hash {
    size_t operator()(uint64_t x) const {
        static const uint64_t FIXED_RANDOM = chrono::steady_clock::now().time_since_epoch().count();
        return x + FIXED_RANDOM;
    }
};
```

See [my post on making randomized solutions unhackable](https://codeforces.com/blog/entry/61675) for more details. Awesome, so our hash is perfectly safe now, right? Not so fast. All we've done is add the same fixed number to every input to the function. But if two numbers a and b satisfy a = b (mod m), then a + x = b + x (mod m) for every x as well. Similar problems occur for other very simple hash functions: multiplying by a random large odd number (and overflowing mod 2⁶⁴) is likely effectively modulo p, but will be problematic for `gp_hash_table`'s power of two policy; the same situation occurs for xor-ing with a random number.

A slightly better hash function like the following may look enticing:

```cpp
struct custom_hash {
    size_t operator()(uint64_t x) const {
        static const uint64_t FIXED_RANDOM = chrono::steady_clock::now().time_since_epoch().count();
        x ^= FIXED_RANDOM;
        return x ^ (x >> 16);
    }
};
```

However, if you are using a `gp_hash_table` this actually still leaves you susceptible to hacks from a strong enough adversary. In particular, after inserting the numbers `(1 << 16) + 1`, `(2 << 16) + 2`, `(3 << 16) + 3`, ..., into this hash table, all of the outputs will be equivalent modulo 2¹⁶. (Do you see why?)

So we want a better hash function, ideally one where changing any input bit results in a 50-50 chance to change any output bit. Note for example that in the hash function `x + FIXED_RANDOM`, this property is not satisfied at all; for example, changing a higher bit in `x` results in a 0% chance of changing a lower bit of the output.

Personally, I like to use [splitmix64](http://xoshiro.di.unimi.it/splitmix64.c), which is extremely high-quality and fast; credit goes to Sebastiano Vigna for designing it. Adding all this together, we have our safe custom hash function:

```cpp
struct custom_hash {
    static uint64_t splitmix64(uint64_t x) {
        // http://xorshift.di.unimi.it/splitmix64.c
        x += 0x9e3779b97f4a7c15;
        x = (x ^ (x >> 30)) * 0xbf58476d1ce4e5b9;
        x = (x ^ (x >> 27)) * 0x94d049bb133111eb;
        return x ^ (x >> 31);
    }

    size_t operator()(uint64_t x) const {
        static const uint64_t FIXED_RANDOM = chrono::steady_clock::now().time_since_epoch().count();
        return splitmix64(x + FIXED_RANDOM);
    }
};
```

Now we can simply define our `unordered_map` or our `gp_hash_table` as follows:

```cpp
unordered_map<long long, int, custom_hash> safe_map;
gp_hash_table<long long, int, custom_hash> safe_hash_table;
```

Once we use these in our program above, it runs very quickly:

```cpp
x = 107897: 0.035 seconds, sum = 2666686666700000
x = 126271: 0.031 seconds, sum = 2666686666700000

=====
Used: 109 ms, 9204 KB
```
