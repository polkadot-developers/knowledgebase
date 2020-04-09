---
slug: storage
lang: en
title: Runtime Storage
---

Runtime storage allows you to store data in your blockchain that is persisted between blocks and can be accessed from within your runtime logic. Storage should be one of the most critical concerns of a blockchain runtime developer. This statement is somewhat self-evident, since one of the primary objectives of a blockchain is to provide decentralized consensus about the state of the underlying storage. Furthermore, well designed storage systems reduce the load on nodes in the network, which will lower the overhead for participants in your blockchain. Substrate exposes a set of layered, modular storage APIs that allow runtime developers to make the storage decisions that suit them best. However, the fundamental principal of blockchain runtime storage is to minimize its use. This document is intended to provide information and best practices about Substrate's runtime storage interfaces. Please refer to [the advanced storage documentation](../advanced/storage) for more information about how these interfaces are implemented.

## Storage Items

[The `storage` module in the Substrate Support pallet](https://substrate.dev/rustdocs/master/frame_support/storage/index.html) gives runtime developers access to Substrate's flexible storage APIs:

* [Storage Value](https://substrate.dev/rustdocs/master/frame_support/storage/trait.StorageValue.html) - A single value
* [Storage Map](https://substrate.dev/rustdocs/master/frame_support/storage/trait.StorageMap.html) - A key-value hash map
* [Storage Double Map](https://substrate.dev/rustdocs/master/frame_support/storage/trait.StorageDoubleMap.html) - An implementation of a map with two keys that
  provides the important ability to efficiently remove all entries that have a common first key
* Iterable Storage Maps - Implementations of [Storage Map](https://substrate.dev/rustdocs/master/frame_support/storage/trait.IterableStorageMap.html) and [Storage Double Map](https://substrate.dev/rustdocs/master/frame_support/storage/trait.IterableStorageDoubleMap.html) whose keys and values can be iterated over

Any value which can be encoded by the [Parity SCALE codec](../advanced/codec) is supported by these
storage APIs.

The type of storage item you use should depend on the logical way in which the value will be used by your runtime.

### Storage Value

This type of storage item should be used for values that are viewed as a single unit by the runtime, whether that is a single primitive value, a single `struct` or a single collection of related items. Although wrapping related items in a shared `struct` is an excellent way to reduce the number of storage reads (an important consideration), at some point the size of the object will begin to incur costs that may outweigh the optimization in storage reads. Storage values can be used to store lists of items, but runtime developers should take care with respect to the size of these lists. Large lists incur storage costs just like large `structs`. Furthermore, iterating over a large list in your runtime may result in exceeding the block production time.

### Storage Maps

Substrate maps are implemented as hash maps, which is a pattern that should be familiar to most developers. In order to give blockchain engineers increased control over the way in which these data structures are stored, Substrate allows developers to select the hashing algorithm that is used to generate map keys. Map data structures are ideal for managing sets of items whose elements will be accessed randomly, as opposed to iterating over them sequentially in their entirety.

### Iterable Storage Maps

The Substrate storage API provides iterable map implementations. Because maps are often used to track unbounded sets of data (account balances, for example) it is especially likely to exceed block production time by iterating over maps in their entirety within the runtime. Furthermore, because maps are comprised of more layers of indirection than native lists, they are significantly more costly than lists to iterate over with respect to time. Depending on the hashing algorithm that you select to generate a map's keys, you may be able to iterate across its keys as well as its values.

## Declaring Storage Items

You can use the `decl_storage!` macro to easily create new runtime storage items. Here is an example of what it looks like to declare each type of storage item:

```rust
decl_storage! {
	trait Store for Module<T: Trait> as Example {
		pub SomeValue: u64;
		pub SomeMap: map u64 => u64;
		pub SomeLinkedMap: linked_map u64 => u64;
		pub SomeDoubleMap: double_map u64, blake2_256(u64) => u64;
	}
}
```

Notice that the last item, the `double_map` specifies the "hasher" (the hash algorithm) that should be used to generate one of its key. Keep reading for more information about the different hashing algorithms and when to use them.

### Default Values

Substrate allows you to specify a default value that is returned when a storage item's value is not set. The default value does **not** actually occupy runtime storage, but runtime logic will see this value during execution.

Here is an example of specifying the default value for all items in a map:

```rust
decl_storage! {
	trait Store for Module<T: Trait> as Example {
		pub SomeMap: map u64 => u64 = 1337;
	}
}
```

## Hashing Algorithms

As mentioned above, a novel feature of Substrate's storage maps is that they allow developers to specify the hashing algorithm that will be used when generating a map's keys. A Rust object that is used to encapsulate hashing logic is referred to as a "hasher". Broadly speaking, the hashers that are available to Substrate developers can be described in two ways: whether or not they are cryptographic and whether or not they produce output that is transparent. In this document you will find information about the pros and cons of the different approaches to hashing; read the advanced storage document if you would like a better understanding of _why_ the different algorithms may behave differently.

### Cryptographic Hashing Algorithms

Cryptographic hashing algorithms are those that use cryptography to make it challenging to use the input to the hashing algorithm to influence its output. For example, a cryptographic hashing algorithm would produce a wide distribution of outputs even if the inputs were the numbers 1 through 10. It is critical to use cryptographic hashing algorithms when users are able to influence the keys of a storage map. Failure to do so creates an attack vector that makes it easy for malicious actors to degrade the performance of your blockchain network. An example of a map that should use a cryptographic hash algorithm to generate its keys is a map used to track account balances. In this case, it is important to use a cryptographic hashing algorithm so that an attacker cannot bombard your system with many small transfers to sequential account numbers; without a cryptographic hash algorithm this would create an imbalanced storage structure that would suffer in performance. Cryptographic hashing algorithms are more complex and resource-intensive than their non-cryptographic counterparts, which is why Substrate allows developers to select when they are used.

### Transparent Hashing Algorithms

A transparent hashing algorithm is one that makes it easy to discover and verify the input that was used to generate a given output. In Substrate, hashing algorithms are made transparent by concatenating the algorithm's input to its output. This makes it trivial for users to retrieve a key's original unhashed value and verify it (by re-hashing it) if they'd like. It is generally recommended to use transparent hashing algorithms for your runtime's storage maps. In fact, it is necessary to use a transparent hashing algorithm if you would like to iterate over the keys in a map.

### Common Substrate Hashers

This table lists some common hashers used in Substrate and describes which ones are cryptographic and which ones are transparent:

| Hasher                                                                                                | Cryptographic | Transparent |
| ----------------------------------------------------------------------------------------------------- | ------------- | ----------- |
| [Blake2 128](https://substrate.dev/rustdocs/master/frame_support/struct.Blake2_128.html)              | X             |             |
| [TwoX 128](https://substrate.dev/rustdocs/master/frame_support/struct.Twox128.html)                   |               |             |
| [Blake2 128 Concat](https://substrate.dev/rustdocs/master/frame_support/struct.Blake2_128Concat.html) | X             | X           |
| [TwoX 64 Concat](https://substrate.dev/rustdocs/master/frame_support/struct.Twox64Concat.html)        |               | X           |
| [Identity](https://substrate.dev/rustdocs/master/frame_support/struct.Identity.html)                  |               |             |

The Identity hasher exposes a hashing algorithm that has an output equal to its input (the identity function).This type of hasher should only be used when the starting key is already a cryptographic hash.

## Child Storage Tries

TODO

## Storage Cache

TODO

## Verify First, Write Last

TODO

## Next Steps

### Learn More

TODO

### Examples

- View this example to see how you can use a `double_map` to act as a `killable` single-map.

### References

- Visit the reference docs for the
  [`decl_storage!` macro](https://substrate.dev/rustdocs/master/frame_support/macro.decl_storage.html)
  more details possible storage declarations.

- Visit the reference docs for
  [StorageValue](https://substrate.dev/rustdocs/master/frame_support/storage/trait.StorageValue.html),
  [StorageMap](https://substrate.dev/rustdocs/master/frame_support/storage/trait.StorageMap.html),
  [StorageLinkedMap](https://substrate.dev/rustdocs/master/frame_support/storage/trait.StorageLinkedMap.html),
  and
  [StorageDoubleMap](https://substrate.dev/rustdocs/master/frame_support/storage/trait.StorageDoubleMap.html)
  to learn more about their API.
