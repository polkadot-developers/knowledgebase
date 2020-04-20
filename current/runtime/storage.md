---
slug: storage
lang: en
title: Runtime Storage
---

Runtime storage allows you to store data in your blockchain that is persisted between blocks and can be accessed from
within your runtime logic. Storage should be one of the most critical concerns of a blockchain runtime developer. This
statement is somewhat self-evident, since one of the primary objectives of a blockchain is to provide decentralized
consensus about the state of the underlying storage. Furthermore, well designed storage systems reduce the load on
nodes in the network, which will lower the overhead for participants in your blockchain. Substrate exposes a set of
layered, modular storage APIs that allow runtime developers to make the storage decisions that suit them best. However,
the fundamental principal of blockchain runtime storage is to minimize its use. This document is intended to provide
information and best practices about Substrate's runtime storage interfaces. Please refer to
[the advanced storage documentation](../advanced/storage) for more information about how these interfaces are implemented.

## Storage Items

[The `storage` module in the Substrate Support pallet](https://substrate.dev/rustdocs/master/frame_support/storage/index.html)
gives runtime developers access to Substrate's flexible storage APIs. Any value which can be encoded by the
[Parity SCALE codec](../advanced/codec) is supported by these storage APIs:

* [Storage Value](https://substrate.dev/rustdocs/master/frame_support/storage/trait.StorageValue.html) - A single value
* [Storage Map](https://substrate.dev/rustdocs/master/frame_support/storage/trait.StorageMap.html) - A key-value hash map
* [Storage Double Map](https://substrate.dev/rustdocs/master/frame_support/storage/trait.StorageDoubleMap.html) - An
  implementation of a map with two keys that provides the important ability to efficiently remove all entries that have
  a common first key
* Iterable Storage Maps - Implementations of
  [Storage Map](https://substrate.dev/rustdocs/master/frame_support/storage/trait.IterableStorageMap.html) and
  [Storage Double Map](https://substrate.dev/rustdocs/master/frame_support/storage/trait.IterableStorageDoubleMap.html)
  whose keys and values can be iterated over

The type of Storage Item you select should depend on the logical way in which the value will be used by your runtime.

### Storage Value

This type of storage item should be used for values that are viewed as a single unit by the runtime, whether that is a
single primitive value, a single `struct` or a single collection of related items. Although wrapping related items in
a shared `struct` is an excellent way to reduce the number of storage reads (an important consideration), at some point
the size of the object will begin to incur costs that may outweigh the optimization in storage reads. Storage values
can be used to store lists of items, but runtime developers should take care with respect to the size of these lists.
Large lists incur storage costs just like large `structs`. Furthermore, iterating over a large list in your runtime may
result in exceeding the block production time.

#### Methods

Refer to the Storage Value documentation for
[a comprehensive list of the methods that Storage Values expose](https://substrate.dev/rustdocs/master/frame_support/storage/trait.StorageValue.html#required-methods).
Some of the most important methods are summarized here:

* [`get()`](https://substrate.dev/rustdocs/master/frame_support/storage/trait.StorageValue.html#tymethod.get) - Load
  the value from storage.
* [`put(val)`](https://substrate.dev/rustdocs/master/frame_support/storage/trait.StorageValue.html#tymethod.put) - Store
  the provided value.
* [`mutate(fn)`](https://substrate.dev/rustdocs/master/frame_support/storage/trait.StorageValue.html#tymethod.mutate) -
  Mutate the value with the provided function.
* [`take()`](https://substrate.dev/rustdocs/master/frame_support/storage/trait.StorageValue.html#tymethod.take) - Remove
  the value from storage.

### Storage Maps

Storage Maps are implemented as maps with hashed keys, which is a pattern that should be familiar to most developers.
Unlike traditional hash maps, though, [Storage Maps in Substrate are simple key-value stores](../advanced/storage) that
do not take key collision into account. The hashing algorithms that Substrate supplies are designed so that runtime
developers do not need to worry about key collisions, but the implementation of Substrate Storage Maps does becomes
important when [querying storage](#Querying-Storage) to read the elements of a Storage Map. In order to give blockchain
engineers increased control over the way in which these data structures are stored, Substrate allows developers to select
the hashing algorithm that is used to generate map keys. Map data structures are ideal for managing sets of items whose
elements will be accessed randomly, as opposed to iterating over them sequentially in their entirety.

#### Methods

[Storage Maps expose an API](https://substrate.dev/rustdocs/master/frame_support/storage/trait.StorageMap.html#required-methods)
that is similar to that of Storage Values.

* `get` - Load the value associated with the provided key from storage. Docs:
  [`StorageMap#get(key)`](https://substrate.dev/rustdocs/master/frame_support/storage/trait.StorageMap.html#tymethod.get),
	[`StorageDoubleMap#get(key1, key2)`](https://substrate.dev/rustdocs/master/frame_support/storage/trait.StorageDoubleMap.html#tymethod.get)
* `insert` - Store the provided value by associating it with the given key. Docs:
  [`StorageMap#insert(key, val)`](https://substrate.dev/rustdocs/master/frame_support/storage/trait.StorageMap.html#tymethod.insert),
	[`StorageDoubleMap#insert(key1, key2, val)`](https://substrate.dev/rustdocs/master/frame_support/storage/trait.StorageDoubleMap.html#tymethod.insert)
* `mutate` - Use the provided function to mutate the value associated with the given key. Docs:
  [`StorageMap#mutate(key, fn)`](https://substrate.dev/rustdocs/master/frame_support/storage/trait.StorageMap.html#tymethod.mutate),
	[`StorageDoubleMap#mutate(key1, key2, fn)`](https://substrate.dev/rustdocs/master/frame_support/storage/trait.StorageDoubleMap.html#tymethod.mutate)
* `take` - Remove the value associated with the given key from storage. Docs:
  [`StorageMap#take(key)`](https://substrate.dev/rustdocs/master/frame_support/storage/trait.StorageMap.html#tymethod.take),
	[`StorageDoubleMap#take(key1, key2)`](https://substrate.dev/rustdocs/master/frame_support/storage/trait.StorageDoubleMap.html#tymethod.take)

### Iterable Storage Maps

The Substrate storage API provides iterable map implementations. Because maps are often used to track unbounded sets of
data (account balances, for example) it is especially likely to exceed block production time by iterating over maps in
their entirety within the runtime. Furthermore, because maps are comprised of more layers of indirection than native
lists, they are significantly more costly than lists to iterate over with respect to time. This is not to say that it
is "wrong" to iterate over maps in your runtime; in general Substrate focuses on "first principals" as opposed to hard
and fast rules of right and wrong. Being efficient within the runtime of a blockchain is an important first principal
of Substrate and this information is designed to help you understand _all_ of Substrate's storage capabilities and use
them in a way that respects the important first principals around which they were designed. Depending on
[the hashing algorithm](#Transparent-Hashing-Algorithms) that you select to generate a map's keys, you may be able to
iterate across its keys as well as its values.

#### Methods

[Iterable Storage Maps expose the following methods](https://substrate.dev/rustdocs/master/frame_support/storage/trait.IterableStorageMap.html#required-methods)
in addition to the other map methods. Note that for Iterable Storage Double Maps, the `iter` and `drain` methods require a parameter, i.e. the first key:

* `iter` - Enumerate all elements in the map in no particular order. If you alter the map while doing this, you'll get undefined
  results. Docs:
	[IterableStorageMap#iter()](https://substrate.dev/rustdocs/master/frame_support/storage/trait.IterableStorageMap.html#tymethod.iter),
	[IterableStorageDoubleMap#iter(key1)](https://substrate.dev/rustdocs/master/frame_support/storage/trait.IterableStorageDoubleMap.html#tymethod.iter)
* `drain` - Remove all elements from the map and iterate through them in no particular order. If you add elements to the map while
  doing this, you'll get undefined results. Docs:
	[IterableStorageMap#drain()](https://substrate.dev/rustdocs/master/frame_support/storage/trait.IterableStorageMap.html#tymethod.drain), [IterableStorageDoubleMap#drain(key1)](https://substrate.dev/rustdocs/master/frame_support/storage/trait.IterableStorageDoubleMap.html#tymethod.drain)
* `translate` - Use the provided function to translate all elements of the map, in no particular order. To remove an element from the
  map, return `None` from the translation function. Docs:
	[IterableStorageMap#translate(fn)](https://substrate.dev/rustdocs/master/frame_support/storage/trait.IterableStorageMap.html#tymethod.translate),
	[IterableStorageDoubleMap#translate(fn)](https://substrate.dev/rustdocs/master/frame_support/storage/trait.IterableStorageDoubleMap.html#tymethod.translate)

## Declaring Storage Items

You can use [the `decl_storage` macro](https://substrate.dev/rustdocs/master/frame_support/macro.decl_storage.html) to
easily create new runtime storage items. Here is an example of what it looks like to declare each type of storage item:

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

Notice that the last item (the `double_map`) specifies the "hasher" (the hash algorithm) that should be used to generate
one of its key. Keep reading for [more information about the different hashing algorithms](#Hashing-Algorithms) and when
to use them.

### Custom Getter Method Names

The `decl_storage` macro makes it easy to provide a custom name for a Storage Item's `get()` method.

Here is an example of renaming the `get()` function of a Storage Value named `SomeValue` to `some_value()`:

```rust
decl_storage! {
	trait Store for Module<T: Trait> as Example {
		pub SomeValue get(some_value): u64;
	}
}
```

### Default Values

Substrate allows you to specify a default value that is returned when a storage item's value is not set. The default
value does **not** actually occupy runtime storage, but runtime logic will see this value during execution.

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

## Hashing Algorithms

As mentioned above, a novel feature of Substrate Storage Maps is that they allow developers to specify the hashing
algorithm that will be used when generating a map's keys. A Rust object that is used to encapsulate hashing logic is
referred to as a "hasher". Broadly speaking, the hashers that are available to Substrate developers can be described
in two ways: whether or not they are cryptographic and whether or not they produce output that is transparent. In this
document you will find information about the pros and cons of the different approaches to hashing; read
[the advanced storage document](../advanced/storage) if you would like a better understanding of _why_ the different
algorithms may behave differently.

### Cryptographic Hashing Algorithms

Cryptographic hashing algorithms are those that use cryptography to make it challenging to use the input to the hashing
algorithm to influence its output. For example, a cryptographic hashing algorithm would produce a wide distribution of
outputs even if the inputs were the numbers 1 through 10. It is critical to use cryptographic hashing algorithms when
users are able to influence the keys of a Storage Map. Failure to do so creates an attack vector that makes it easy for
malicious actors to degrade the performance of your blockchain network. An example of a map that should use a cryptographic
hash algorithm to generate its keys is a map used to track account balances. In this case, it is important to use a
cryptographic hashing algorithm so that an attacker cannot bombard your system with many small transfers to sequential
account numbers; without a cryptographic hash algorithm this would create an imbalanced storage structure that would
suffer in performance. Cryptographic hashing algorithms are more complex and resource-intensive than their non-cryptographic
counterparts, which is why Substrate allows developers to select when they are used.

### Transparent Hashing Algorithms

A transparent hashing algorithm is one that makes it easy to discover and verify the input that was used to generate a
given output. In Substrate, hashing algorithms are made transparent by concatenating the algorithm's input to its output.
This makes it trivial for users to retrieve a key's original unhashed value and verify it if they'd like (by re-hashing
it). It is generally recommended to use transparent hashing algorithms for your runtime's Storage Maps. In fact, it is
necessary to use a transparent hashing algorithm if you would like to iterate over the keys in a map.

### Common Substrate Hashers

This table lists some common hashers used in Substrate and denotes those that are cryptographic and those that are
transparent:

| Hasher                                                                                                | Cryptographic | Transparent |
| ----------------------------------------------------------------------------------------------------- | ------------- | ----------- |
| [Blake2 128](https://substrate.dev/rustdocs/master/frame_support/struct.Blake2_128.html)              | X             |             |
| [TwoX 128](https://substrate.dev/rustdocs/master/frame_support/struct.Twox128.html)                   |               |             |
| [Blake2 128 Concat](https://substrate.dev/rustdocs/master/frame_support/struct.Blake2_128Concat.html) | X             | X           |
| [TwoX 64 Concat](https://substrate.dev/rustdocs/master/frame_support/struct.Twox64Concat.html)        |               | X           |
| [Identity](https://substrate.dev/rustdocs/master/frame_support/struct.Identity.html)                  |               |             |

The Identity hasher encapsulates a hashing algorithm that has an output equal to its input (the identity function).This type
of hasher should only be used when the starting key is already a cryptographic hash.

## Querying Storage

Blockchains that are built with Substrate expose a remote procedure call (RPC) server that can be used to query your
blockchain's runtime storage. One thing that will be important to understand before proceeding is that the Substrate
runtime storage APIs are layers of abstraction that have been built upon a simple key-value database. When you use the
Substrate RPC to query runtime storage, you only need to provide a single piece of information: the storage key. The rest
of this section will go over the basics of how to calculate storage keys for the different types of Storage Items. If
you'd like to learn more about how Substrate uses a key-value database to implement the different kinds of Storage Items,
refer to [the advanced storage documentation](../advanced/storage).

### Storage Value Keys

To calculate the key for a simple Storage Value, take the TwoX 128 hash of the name of the module that contains the
Storage Value and append to it the TwoX 128 hash of the name of the Storage Value itself. For example, the
[Sudo](https://substrate.dev/rustdocs/master/pallet_sudo/index.html) module exposes a Storage Value item named
[`Key`](https://substrate.dev/rustdocs/master/pallet_sudo/struct.Module.html#method.key):

```
twox_128("Sudo")                   = "0x5c0d1176a568c1f92944340dbfed9e9c"
twox_128("Key)                     = "0x530ebca703c85910e7164cb7d1c9e47b"
twox_128("Sudo") + twox_128("Key") = "0x5c0d1176a568c1f92944340dbfed9e9c530ebca703c85910e7164cb7d1c9e47b"
```

If the familiar `Alice` account is the sudo user, an RPC request and response to read the Sudo module's `Key` Storage
Value could be represented as:

```
state_getStorage("0x5c0d1176a568c1f92944340dbfed9e9c530ebca703c85910e7164cb7d1c9e47b") = "0xd43593c715fdd31c61141abd04a99fd6822c8558854ccde39a5684e7a56da27d"
```

In this case, the value that is returned (`"0xd43593c715fdd31c61141abd04a99fd6822c8558854ccde39a5684e7a56da27d"`) is
Alice's [SCALE](../advanced/codec)-encoded account ID (`5GrwvaEF5zXb26Fz9rcQpDWS57CtERHpNehXCPcNoHGKutQY`).

You may have noticed that the non-cryptographic TwoX 128 hash algorithm is used to generate Storage Value keys. This is
because it is not necessary to pay the performance costs associated with a cryptographic hash function since the input
to the hash function (the names of the module and storage item) are determined by you, the runtime developer, and not by
potentially malicious users of your blockchain.

### Storage Map Keys

Like Storage Values, the keys for Storage Maps are equal to the TwoX 128 hash of the name of the module that contains
the map prepended to the TwoX 128 hash of the name of the Storage Map itself. To retrieve an element from a map, simply
append the hash of the desired map key to the storage key of the Storage Map. For maps with two keys (Storage Double
Maps), append the hash of the first map key followed by the hash of the second map key to the Storage Double Map's storage
key. Remember, Substrate will use the TwoX 128 hashing algorithm for the module and storage item names, but you will need
to make sure to use the correct hashing algorithm (the one that was declared in
[the `decl_storage` macro](#Declaring-Storage-Items)) when determining the hashed keys for the elements in a map.

Here is an example that illustrates querying a Storage Map named `FreeBalance` from a module named "Balances" for the
balance of the familiar `Alice` account. In this example, the `FreeBalance` map is using
[the transparent Blake2 128 Concat hashing algorithm](#Transparent-Hashing-Algorithms):

```
twox_128("Balances)                                              = "0xc2261276cc9d1f8598ea4b6a74b15c2f"
twox_128("FreeBalance")                                          = "0x6482b9ade7bc6657aaca787ba1add3b4"
scale_encode("5GrwvaEF5zXb26Fz9rcQpDWS57CtERHpNehXCPcNoHGKutQY") = "0xd43593c715fdd31c61141abd04a99fd6822c8558854ccde39a5684e7a56da27d"

blake2_128_concat("0xd43593c715fdd31c61141abd04a99fd6822c8558854ccde39a5684e7a56da27d") = "0xde1e86a9a8c739864cf3cc5ec2bea59fd43593c715fdd31c61141abd04a99fd6822c8558854ccde39a5684e7a56da27d"

state_getStorage("0xc2261276cc9d1f8598ea4b6a74b15c2f6482b9ade7bc6657aaca787ba1add3b4de1e86a9a8c739864cf3cc5ec2bea59fd43593c715fdd31c61141abd04a99fd6822c8558854ccde39a5684e7a56da27d") = "0x0000a0dec5adc9353600000000000000"
```

The value that is returned from the storage query (`"0x0000a0dec5adc9353600000000000000"` in the example above) is the
[SCALE](../advanced/codec)-encoded value of Alice's account balance (`"1000000000000000000000"` in this example). Notice
that before hashing Alice's account ID it has to be SCALE-encoded. Also notice that the output of the `blake2_128_concat`
function consists of 32 hexadecimal characters followed by the function's input. This is because the Blake2 128 Concat
is [a transparent hashing algorithm as describe above](#Transparent-Hashing-Algorithms). Although the above example may
make this characteristic seem superfluous, its utility becomes more apparent when the goal is to iterate over the keys
in a map (as opposed to retrieving the value associated with a single key). The ability to iterate over the keys in a
map is a common requirement in order to allow _people_ to use the map in a way that seems natural (such as UIs): first,
a user is presented with a list of elements in the map, then, that user can select the element that they are interested
in and query the map for more details about that particular element. Here is another example that uses the same example
Storage Map (a map named `FreeBalances` that uses a Blake2 128 Concat hashing algorithm in a module named "Balances")
that will demonstrate using the Substrate RPC to query a Storage Map for its list of keys via the `state_getKeys` RPC
endpoint:

```
twox_128("Balances)                                       = "0xc2261276cc9d1f8598ea4b6a74b15c2f"
twox_128("FreeBalance")                                   = "0x6482b9ade7bc6657aaca787ba1add3b4"

state_getKeys("0xc2261276cc9d1f8598ea4b6a74b15c2f6482b9ade7bc6657aaca787ba1add3b4") = [
	"0xc2261276cc9d1f8598ea4b6a74b15c2f6482b9ade7bc6657aaca787ba1add3b4de1e86a9a8c739864cf3cc5ec2bea59fd43593c715fdd31c61141abd04a99fd6822c8558854ccde39a5684e7a56da27d",
	"0xc2261276cc9d1f8598ea4b6a74b15c2f6482b9ade7bc6657aaca787ba1add3b432a5935f6edc617ae178fef9eb1e211fbe5ddb1579b72e84524fc29e78609e3caf42e85aa118ebfe0b0ad404b5bdd25f",
	...
]
```

Each element in the list that is returned by the Substrate RPC's `state_getKeys` endpoint can be directly used as input
for the RPC's `state_getStorage` endpoint. In fact, the first element in the example list above is equal to the input
used for the `state_getStorage` query in the previous example (the one used to find the balance for `Alice`). Because
the map that these keys belong to uses a transparent hashing algorithm to generate its keys, it is possible to determine
the account associated with the second element in the list. Notice that each element in the list is a hexadecimal value
that begins with the same 64 characters; this is because each list element represents a key in the same map, and that
map is identified by concatenating two TwoX 128 hashes, each of which are 128-bits or 32 hexadecimal characters. After
discarding this portion of the second element in the list, you are left with
`0x32a5935f6edc617ae178fef9eb1e211fbe5ddb1579b72e84524fc29e78609e3caf42e85aa118ebfe0b0ad404b5bdd25f`. You saw in the
previous example that this represents the Blake2 128 Concat hash of some [SCALE](../advanced/codec)-encoded account ID.
As described above, the Blake 128 Concat hashing algorithm consists of appending (concatenating) the hashing algorithm's
input to its Blake 128 hash. This means that the first 128 bits (or 32 hexadecimal characters) of a Blake2 128 Concat
hash represents a Blake2 128 hash, and the remainder represents the value that was passed to the Blake 2 128 hashing
algorithm. In this example, after you remove the first 32 hexadecimal characters that represent the Blake2 128 hash (i.e.
`0x32a5935f6edc617ae178fef9eb1e211f`) what is left is the hexadecimal value
`0xbe5ddb1579b72e84524fc29e78609e3caf42e85aa118ebfe0b0ad404b5bdd25f`, which is a [SCALE](../advanced/codec)-encoded
account ID. Decoding this value yields the result `5GNJqTPyNqANBkUVMN1LPPrxXnFouWXoe2wNSmmEoLctxiZY`, which is the account
ID for the familiar `Alice_Stash` account.

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

* Visit the reference docs for the
  [`decl_storage!` macro](https://substrate.dev/rustdocs/master/frame_support/macro.decl_storage.html) more details
  possible storage declarations.

* Visit the reference docs for
  [StorageValue](https://substrate.dev/rustdocs/master/frame_support/storage/trait.StorageValue.html),
  [StorageMap](https://substrate.dev/rustdocs/master/frame_support/storage/trait.StorageMap.html),
  [StorageLinkedMap](https://substrate.dev/rustdocs/master/frame_support/storage/trait.StorageLinkedMap.html), and
  [StorageDoubleMap](https://substrate.dev/rustdocs/master/frame_support/storage/trait.StorageDoubleMap.html) to learn
  more about their APIs.
