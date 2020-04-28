---
slug: storage
lang: en
title: Runtime Storage
---

Runtime storage allows you to store data in your blockchain that is persisted between blocks and can
be accessed from within your runtime logic. Storage should be one of the most critical concerns of a
blockchain runtime developer. This statement is somewhat self-evident, since one of the primary
objectives of a blockchain is to provide decentralized consensus about the state of the underlying
storage. Furthermore, well designed storage systems reduce the load on nodes in the network, which
will lower the overhead for participants in your blockchain. Substrate exposes a set of layered,
modular storage APIs that allow runtime developers to make the storage decisions that suit them
best. However, the fundamental principle of blockchain runtime storage is to minimize its use. This
document is intended to provide information and best practices about Substrate's runtime storage
interfaces. Please refer to [the advanced storage documentation](../advanced/storage) for more
information about how these interfaces are implemented.

## Storage Items

The `storage` module in
[FRAME Support](https://substrate.dev/rustdocs/master/frame_support/storage/index.html) gives
runtime developers access to Substrate's flexible storage APIs. Any value that can be encoded by the
[Parity SCALE codec](../advanced/codec) is supported by these storage APIs:

- [Storage Value](https://substrate.dev/rustdocs/master/frame_support/storage/trait.StorageValue.html) -
  A single value
- [Storage Map](https://substrate.dev/rustdocs/master/frame_support/storage/trait.StorageMap.html) -
  A key-value hash map
- [Storage Double Map](https://substrate.dev/rustdocs/master/frame_support/storage/trait.StorageDoubleMap.html) -
  An implementation of a map with two keys that provides the important ability to efficiently remove
  all entries that have a common first key

The type of Storage Item you select should depend on the logical way in which the value will be used
by your runtime.

### Storage Value

This type of storage item should be used for values that are viewed as a single unit by the runtime,
whether that is a single primitive value, a single `struct`, or a single collection of related
items. Although wrapping related items in a shared `struct` is an excellent way to reduce the number
of storage reads (an important consideration), at some point the size of the object will begin to
incur costs that may outweigh the optimization in storage reads. Storage values can be used to store
lists of items, but runtime developers should take care with respect to the size of these lists.
Large lists incur storage costs just like large `structs`. Furthermore, iterating over a large list
in your runtime may result in exceeding the block production time - if this occurs your blockchain
will stop producing blocks, which means that it will stop functioning.

#### Methods

Refer to the Storage Value documentation for
[a comprehensive list of the methods that Storage Values expose](https://substrate.dev/rustdocs/master/frame_support/storage/trait.StorageValue.html#required-methods).
Some of the most important methods are summarized here:

- [`get()`](https://substrate.dev/rustdocs/master/frame_support/storage/trait.StorageValue.html#tymethod.get) -
  Load the value from storage.
- [`put(val)`](https://substrate.dev/rustdocs/master/frame_support/storage/trait.StorageValue.html#tymethod.put) -
  Store the provided value.
- [`mutate(fn)`](https://substrate.dev/rustdocs/master/frame_support/storage/trait.StorageValue.html#tymethod.mutate) -
  Mutate the value with the provided function.
- [`take()`](https://substrate.dev/rustdocs/master/frame_support/storage/trait.StorageValue.html#tymethod.take) -
  Remove the value from storage.

### Storage Maps

Map data structures are ideal for managing sets of items whose elements will be accessed randomly,
as opposed to iterating over them sequentially in their entirety. Storage Maps in Substrate are
implemented as key-value hash maps, which is a pattern that should be familiar to most developers.
In order to give blockchain engineers increased control, Substrate allows developers to select
[the hashing algorithm](#Hashing-Algorithms) that is used to generate a map's keys. Refer to
[the advanced storage documentation](../advanced/storage) to learn more about how Substrate's
Storage Maps are implemented.

#### Methods

[Storage Maps expose an API](https://substrate.dev/rustdocs/master/frame_support/storage/trait.StorageMap.html#required-methods)
that is similar to that of Storage Values.

- `get` - Load the value associated with the provided key from storage. Docs:
  [`StorageMap#get(key)`](https://substrate.dev/rustdocs/master/frame_support/storage/trait.StorageMap.html#tymethod.get),
  [`StorageDoubleMap#get(key1, key2)`](https://substrate.dev/rustdocs/master/frame_support/storage/trait.StorageDoubleMap.html#tymethod.get)
- `insert` - Store the provided value by associating it with the given key. Docs:
  [`StorageMap#insert(key, val)`](https://substrate.dev/rustdocs/master/frame_support/storage/trait.StorageMap.html#tymethod.insert),
  [`StorageDoubleMap#insert(key1, key2, val)`](https://substrate.dev/rustdocs/master/frame_support/storage/trait.StorageDoubleMap.html#tymethod.insert)
- `mutate` - Use the provided function to mutate the value associated with the given key. Docs:
  [`StorageMap#mutate(key, fn)`](https://substrate.dev/rustdocs/master/frame_support/storage/trait.StorageMap.html#tymethod.mutate),
  [`StorageDoubleMap#mutate(key1, key2, fn)`](https://substrate.dev/rustdocs/master/frame_support/storage/trait.StorageDoubleMap.html#tymethod.mutate)
- `take` - Remove the value associated with the given key from storage. Docs:
  [`StorageMap#take(key)`](https://substrate.dev/rustdocs/master/frame_support/storage/trait.StorageMap.html#tymethod.take),
  [`StorageDoubleMap#take(key1, key2)`](https://substrate.dev/rustdocs/master/frame_support/storage/trait.StorageDoubleMap.html#tymethod.take)

#### Iterable Storage Maps

Depending on [the hashing algorithm](#Transparent-Hashing-Algorithms) that you select to generate a
map's keys, you may be able to iterate across its keys and values. Because maps are often used to
track unbounded sets of data (account balances, for example) it is especially likely to exceed block
production time by iterating over maps in their entirety within the runtime. Furthermore, because
maps are comprised of more layers of indirection than native lists, they are significantly more
costly than lists to iterate over with respect to time. This is not to say that it is "wrong" to
iterate over maps in your runtime; in general Substrate focuses on "first principles" as opposed to
hard and fast rules of right and wrong. Being efficient within the runtime of a blockchain is an
important first principle of Substrate and this information is designed to help you understand _all_
of Substrate's storage capabilities and use them in a way that respects the important first
principles around which they were designed.

##### Iterable Storage Map Methods

Substrate's Iterable Storage Map interfaces define the following methods. Note that for Iterable
Storage Double Maps, the `iter` and `drain` methods require a parameter, i.e. the first key:

- `iter` - Enumerate all elements in the map in no particular order. If you alter the map while
  doing this, you'll get undefined results. Docs:
  [IterableStorageMap#iter()](https://substrate.dev/rustdocs/master/frame_support/storage/trait.IterableStorageMap.html#tymethod.iter),
  [IterableStorageDoubleMap#iter(key1)](https://substrate.dev/rustdocs/master/frame_support/storage/trait.IterableStorageDoubleMap.html#tymethod.iter)
- `drain` - Remove all elements from the map and iterate through them in no particular order. If you
  add elements to the map while doing this, you'll get undefined results. Docs:
  [IterableStorageMap#drain()](https://substrate.dev/rustdocs/master/frame_support/storage/trait.IterableStorageMap.html#tymethod.drain),
  [IterableStorageDoubleMap#drain(key1)](https://substrate.dev/rustdocs/master/frame_support/storage/trait.IterableStorageDoubleMap.html#tymethod.drain)
- `translate` - Use the provided function to translate all elements of the map, in no particular
  order. To remove an element from the map, return `None` from the translation function. Docs:
  [IterableStorageMap#translate(fn)](https://substrate.dev/rustdocs/master/frame_support/storage/trait.IterableStorageMap.html#tymethod.translate),
  [IterableStorageDoubleMap#translate(fn)](https://substrate.dev/rustdocs/master/frame_support/storage/trait.IterableStorageDoubleMap.html#tymethod.translate)

#### Hashing Algorithms

As mentioned above, a novel feature of Substrate Storage Maps is that they allow developers to
specify the hashing algorithm that will be used when generating a map's keys. A Rust object that is
used to encapsulate hashing logic is referred to as a "hasher". Broadly speaking, the hashers that
are available to Substrate developers can be described in two ways: whether or not they are
cryptographic and whether or not they produce output that is transparent.

##### Cryptographic Hashing Algorithms

Cryptographic hashing algorithms are those that use cryptography to make it challenging to use the
input to the hashing algorithm to influence its output. For example, a cryptographic hashing
algorithm would produce a wide distribution of outputs even if the inputs were the numbers 1
through 10. It is critical to use cryptographic hashing algorithms when users are able to influence
the keys of a Storage Map. Failure to do so creates an attack vector that makes it easy for
malicious actors to degrade the performance of your blockchain network. An example of a map that
should use a cryptographic hash algorithm to generate its keys is a map used to track account
balances. In this case, it is important to use a cryptographic hashing algorithm so that an attacker
cannot bombard your system with many small transfers to sequential account numbers; without a
cryptographic hash algorithm this would create an imbalanced storage structure that would suffer in
performance. Cryptographic hashing algorithms are more complex and resource-intensive than their
non-cryptographic counterparts, which is why Substrate allows developers to select when they are
used.

##### Transparent Hashing Algorithms

A transparent hashing algorithm is one that makes it easy to discover and verify the input that was
used to generate a given output. In Substrate, hashing algorithms are made transparent by
concatenating the algorithm's input to its output. This makes it trivial for users to retrieve a
key's original unhashed value and verify it if they'd like (by re-hashing it). It is generally
recommended to use transparent hashing algorithms for your runtime's Storage Maps. In fact, it is
_necessary_ to use a transparent hashing algorithm if you would like access
[iterable map](#Iterable-Storage-Maps) capabilities.

##### Common Substrate Hashers

This table lists some common hashers used in Substrate and denotes those that are cryptographic and
those that are transparent:

| Hasher                                                                                                | Cryptographic | Transparent |
| ----------------------------------------------------------------------------------------------------- | ------------- | ----------- |
| [Blake2 128](https://substrate.dev/rustdocs/master/frame_support/struct.Blake2_128.html)              | X             |             |
| [TwoX 128](https://substrate.dev/rustdocs/master/frame_support/struct.Twox128.html)                   |               |             |
| [Blake2 128 Concat](https://substrate.dev/rustdocs/master/frame_support/struct.Blake2_128Concat.html) | X             | X           |
| [TwoX 64 Concat](https://substrate.dev/rustdocs/master/frame_support/struct.Twox64Concat.html)        |               | X           |
| [Identity](https://substrate.dev/rustdocs/master/frame_support/struct.Identity.html)                  |               |             |

The Identity hasher encapsulates a hashing algorithm that has an output equal to its input (the
identity function).This type of hasher should only be used when the starting key is already a
cryptographic hash.

## Declaring Storage Items

You can use
[the `decl_storage` macro](https://substrate.dev/rustdocs/master/frame_support/macro.decl_storage.html)
to easily create new runtime storage items. Here is an example of what it looks like to declare each
type of storage item:

```rust
decl_storage! {
	trait Store for Module<T: Trait> as Example {
		pub SomePrimitiveValue get(fn some_primitive_value): u32;
		// complex types are prefaced by T::
		pub SomeComplexValue: T::AccountId;
		pub SomeMap get(fn some_map): map hasher(blake2_128_concat) str => u32;
		pub SomeDoubleMap: double_map hasher(blake2_128_concat) str, hasher(blake2_128_concat) str => u32;
	}
}
```

Notice that the map storage items specify [the hashing algorithm](#Hashing-Algorithms) that will be
used.

### Getter Methods

The `decl_storage` macro provides an optional `get` extension that can be used to implement a getter
method for a storage item on the module that contains that storage item; the extension takes the
desired name of the getter function as an argument. If you omit this optional extension, you will
still be able to access the storage item's value, but you will not be able to do so by way of a
getter method implemented on the module; instead, you will need to need to use
[the storage item's `get` method](#Methods). Keep in mind that the optional `get` extension only
impacts the way that the storage item can be accessed from within Substrate code; you will always be
able to [query the storage of your runtime](../advanced/storage#Querying-Storage) to get the value
of a storage item.

Here is an example that implements a getter method named `some_value` for a Storage Value named
`SomeValue`. This module would now have access to a `Self::some_value()` method in addition to the
`SomeValue::get()` method:

```rust
decl_storage! {
	trait Store for Module<T: Trait> as Example {
		pub SomeValue get(fn some_value): u64;
	}
}
```

### Default Values

Substrate allows you to specify a default value that is returned when a storage item's value is not
set. The default value does **not** actually occupy runtime storage, but runtime logic will see this
value during execution.

Here is an example of specifying the default value for all items in a map:

```rust
decl_storage! {
	trait Store for Module<T: Trait> as Example {
		pub SomeMap: map u64 => u64 = 1337;
	}
}
```

### Genesis Config

You can define
[an optional `GenesisConfig`](https://substrate.dev/rustdocs/master/frame_support/macro.decl_storage.html#genesisconfig)
struct in order to initialize Storage Items in the genesis block of your blockchain.

// TODO

## Accessing Storage Items

Blockchains that are built with Substrate expose a remote procedure call (RPC) server that can be
used to query your blockchain's runtime storage. You can use software libraries like
[Polkadot JS](https://polkadot.js.org/) to easily interact with the RPC server from your code and
access storage items. The Polkadot JS team also maintains
[the Polkadot Apps UI](https://polkadot.js.org/apps), which is a fully-featured web app for
interacting with Substrate-based blockchains, including querying storage. Refer to
[the advanced storage documentation](../advanced/storage) to learn more about how Substrate uses a
key-value database to implement the different kinds of Storage Items and how to query this database
directly by way of the RPC server.

## Child Storage Tries

TODO

## Storage Cache

TODO

## Best Practices

TODO

### Verify First, Write Last

TODO

## Next Steps

### Learn More

Read [the advanced storage documentation](../advanced/storage).

### Examples

Check out
[the Substrate Recipes section on storage](https://substrate.dev/recipes/3-entrees/storage-api/index.html).

### References

- Visit the reference docs for the
  [`decl_storage!` macro](https://substrate.dev/rustdocs/master/frame_support/macro.decl_storage.html)
  for more details about the available storage declarations.

- Visit the reference docs for
  [StorageValue](https://crates.parity.io/frame_support/storage/trait.StorageValue.html),
  [StorageMap](https://crates.parity.io/frame_support/storage/trait.StorageMap.html) and
  [StorageDoubleMap](https://crates.parity.io/frame_support/storage/trait.StorageDoubleMap.html) to
  learn more about their APIs.
