`pulp` is a safe abstraction over SIMD instructions, that allows you to write a function once
and dispatch to equivalent vectorized versions based on the features detected at runtime.

# Autovectorization example

```rust
use pulp::Arch;

let mut v = (0..1000).map(|i| i as f64).collect::<Vec<_>>();
let arch = Arch::new();

arch.dispatch(|| {
    for x in &mut v {
        *x *= 2.0;
    }
});

for (i, x) in v.into_iter().enumerate() {
    assert_eq!(x, 2.0 * i as f64);
}
```

# Manual vectorization example

```rust
use pulp::{Arch, Simd, WithSimd};

struct TimesThree<'a>(&'a mut [f64]);
impl<'a> WithSimd for TimesThree<'a> {
    type Output = ();

    #[inline(always)]
    fn with_simd<S: Simd>(self, simd: S) -> Self::Output {
        let v = self.0;
        let (head, tail) = S::f64s_as_mut_simd(v);

        let three = simd.f64s_splat(3.0);
        for x in head {
            *x = simd.f64s_mul(three, *x);
        }

        for x in tail {
            *x = *x * 3.0;
        }
    }
}

let mut v = (0..1000).map(|i| i as f64).collect::<Vec<_>>();
let arch = Arch::new();

arch.dispatch(TimesThree(&mut v));

for (i, x) in v.into_iter().enumerate() {
    assert_eq!(x, 3.0 * i as f64);
}
```
