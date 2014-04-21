entropy
=======

## Cosmic Background Radiation

> but i think if you just make a microwave-band, put it in a parabolic reflector, and point it at the sky, some significant portion of the noise will be cosmic background - celoyd

* [Geiger Counter tubes](http://www.ebay.com/itm/Geiger-Muller-tube-counter-SI3BG-NEW-lot-of-10-tubes-/251278027433?pt=BI_Security_Fire_Protection&hash=item3a81566aa9)

## Tests

* [NIST Test](http://csrc.nist.gov/groups/ST/toolkit/rng/documentation_software.html)

## Behind the scenes

### ECMA 262

http://www.ecma-international.org/ecma-262/5.1/#sec-15.8.2.14

> 15.8.2.14 random ( )
> Returns a Number value with positive sign, greater than or equal to 0 but less than 1, chosen randomly or pseudo randomly with approximately uniform distribution over that range, using an implementation-dependent algorithm or strategy. This function takes no arguments.

[V8:Random](https://code.google.com/p/v8/source/browse/trunk/src/v8.cc#112)

```cpp
// Used by JavaScript APIs
uint32_t V8::Random(Context* context) {
  ASSERT(context->IsNativeContext());
  ByteArray* seed = context->random_seed();
  uint32_t* state = reinterpret_cast<uint32_t*>(seed->GetDataStartAddress());

  // When we get here, the RNG must have been initialized,
  // see the Genesis constructor in file bootstrapper.cc.
  ASSERT_NE(0, state[0]);
  ASSERT_NE(0, state[1]);

  // Mix the bits.  Never replaces state[i] with 0 if it is nonzero.
  state[0] = 18273 * (state[0] & 0xFFFF) + (state[0] >> 16);
  state[1] = 36969 * (state[1] & 0xFFFF) + (state[1] >> 16);

  return (state[0] << 14) + (state[1] & 0x3FFFF);
}
```

Upstream

```cpp
DEFINE_int(random_seed, 0,
           "Default seed for initializing random generator "
           "(0, the default, means to use system random).")
```

EntropySource

```cpp

#if V8_OS_CYGWIN || V8_OS_WIN
  // Use rand_s() to gather entropy on Windows. See:
  // https://code.google.com/p/v8/issues/detail?id=2905
  unsigned first_half, second_half;
  errno_t result = rand_s(&first_half);
  ASSERT_EQ(0, result);
  result = rand_s(&second_half);
  ASSERT_EQ(0, result);
  SetSeed((static_cast<int64_t>(first_half) << 32) + second_half);
#else
  // Gather entropy from /dev/urandom if available.
  FILE* fp = fopen("/dev/urandom", "rb");
  if (fp != NULL) {
    int64_t seed;
    size_t n = fread(&seed, sizeof(seed), 1, fp);
    fclose(fp);
    if (n == 1) {
      SetSeed(seed);
      return;
    }
  }

  // We cannot assume that random() or rand() were seeded
  // properly, so instead of relying on random() or rand(),
  // we just seed our PRNG using timing data as fallback.
  // This is weak entropy, but it's sufficient, because
  // it is the responsibility of the embedder to install
  // an entropy source using v8::V8::SetEntropySource(),
  // which provides reasonable entropy, see:
  // https://code.google.com/p/v8/issues/detail?id=2905
  int64_t seed = Time::NowFromSystemTime().ToInternalValue() << 24;
  seed ^= TimeTicks::HighResolutionNow().ToInternalValue() << 16;
  seed ^= TimeTicks::Now().ToInternalValue() << 8;
  SetSeed(seed);
#endif  // V8_OS_CYGWIN || V8_OS_WIN
}
```

### [node](https://github.com/joyent/node/blob/master/src/node.cc#L3345-L3349)

```cpp
#if HAVE_OPENSSL
  // V8 on Windows doesn't have a good source of entropy. Seed it from
  // OpenSSL's pool.
  V8::SetEntropySource(crypto::EntropySource);
#endif
```

### openssl

* http://wiki.openssl.org/index.php/Random_Numbers
* http://research.swtch.com/openssl


### Hardware

* https://en.wikipedia.org/wiki/Hardware_random_number_generator
* https://en.wikipedia.org/wiki/Randomness_extractor
