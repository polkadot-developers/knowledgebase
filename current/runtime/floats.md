---
slug: floats
lang: en
title: Floating Point Numbers
---

When developing for the Substrate runtime and doing calculations it is appealing to use floating point numbers for e.g. representing fractions. In Substrate it is important that each node arrives at the same deterministic result for consensus. This document covers why traditional floating point numbers are problematic in runtime and what alternatives exist.

## Problems with Floating Point Numbers in Runtime

The standard way of handling floating points is dependent on the need for decimal places by using one of the in-built primitive types of handlingfix point arithmetic. Floating point numbers are not deterministic and fixed point computation is safe for Substrate since it represents all rationals as a fraction thus resolving a deterministic result. The two types which are used to handle large fixed point arithmetic are Permill and Perbill, you can refer to [safe math](https://substrate.dev/recipes/3-entrees/safemath.html)for further information and implementation.

## Alternatives

### Perbill and Friends

### substrate-fixed
[substrate-fixed repo](https://github.com/encointer/substrate-fixed)


## Next Steps
**TODO**

### Learn More

- 

### Examples

- [Fixed Point Recipe](https://github.com/substrate-developer-hub/recipes/pull/196)

### References

- View the [primitive types defined in
  `node-primitives`](https://substrate.dev/rustdocs/master/node_primitives/index.html).
  
- View the [`traits` defined in `sp-runtime`](https://substrate.dev/rustdocs/master/sp_runtime/traits/index.html)
