entropy
=======

## Cosmic Background Radiation

> but i think if you just make a microwave-band, put it in a parabolic reflector, and point it at the sky, some significant portion of the noise will be cosmic background - celoyd

* [Geiger Counter tubes](http://www.ebay.com/itm/Geiger-Muller-tube-counter-SI3BG-NEW-lot-of-10-tubes-/251278027433?pt=BI_Security_Fire_Protection&hash=item3a81566aa9)

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
