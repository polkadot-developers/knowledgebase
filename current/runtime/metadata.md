---
slug: metadata
lang: en
title: Runtime Metadata
---

The Substrate Metadata lets you inspect type and error information of calls, events, storage items, 
and constants of every pallet in a Substrate runtime. It's automatically generated from the 
contents within the Substrate macros (`decl_module!`, `decl_storage!`, `decl_event!`, and 
`decl_error!`). With the metadata, you are able to acquire the information necessary to query 
storage and code around a specific runtime's types without importing the code from that runtime, or 
any previous runtime.

Metadata for a specific version of the chain is stored on a per-block basis. Querying the metadata 
for an older block (with an archive node, for example) could result in acquiring an older version 
of metadata that is not compatible with the current runtime. Metadata will include information 
specific to a single runtime rather than being generic over any runtime.

> All examples in this document were taken from block 1,768,321 on Kusama. See the 
  [full metadata](https://gist.github.com/insipx/db5e49c0160b1f1bd421a3c34fefdf48).

## How to Get the Metadata

### Rust

The easiest way to get the metadata is by querying the automatically generated JSON-RPC function 
`state_getMetadata`. This will return a vector of SCALE-encoded bytes. You can decode this using 
the `frame-metadata` and `parity-scale-codec` libraries.

Some helpful libraries like `substrate-subxt` fetch the metadata and decode them for you. Once 
decoded, the structure may be serialized into JSON with serde. If you'd prefer to use the RPC more 
directly, the [JSONRPC](https://github.com/paritytech/jsonrpc) and 
[jsonrpsee](https://github.com/paritytech/jsonrpsee) libraries provide an interface to do so from 
Rust, and both were created with Substrate/Polkadot in mind.

### Javascript

If you are using Javascript, [`polkadot-js/api`](https://polkadot.js.org/api/) already provides 
APIs to interact with a Substrate blockchain, including the 
[`getMetadata`](https://polkadot.js.org/api/METHODS_RPC.html#json-rpc) function.

You can try the following code snippets to fetch the metadata in this 
[Substrate UI](https://polkadot.js.org/apps/#/js) page:

```javascript
const { magicNumber, metadata } = await api.rpc.state.getMetadata();

console.log( 'Magic number: ' + magicNumber );
console.log( 'Metadata: ' + metadata.raw );
```

### Other Languages

You can send a **WebSocket message** or **HTTP POST request** to a Substrate node endpoint by using 
any existing client. The message or body for `getMetadata` is:

```json
{
  "id": 1,
  "jsonrpc": "2.0",
  "method": "state_getMetadata",
  "params": []
}
```

You can leave `params` empty, or if you want to fetch the metadata for a specific block, provide 
the block's hash:

```json
{
  "id": 1,
  "jsonrpc": "2.0",
  "method": "state_getMetadata",
  "params": ["ca15c2f1e1540517697b6b5f2cc6bc0c60876a1a1af604269b7215970798bbed"]
}
```

Where `0xca15c2f1e1540517697b6b5f2cc6bc0c60876a1a1af604269b7215970798bbed` is the hash of 
block 1,768,321.

The response has the following format, but with much more information in the `result` field:

```json
{
    "jsonrpc": "2.0",
    "result": "0x6d65746104241873797374656d1853797374656d012c304163636f756e744e6f6e636501010130543a3a4163636f756e74...",
    "id": 1
}
```

The hexadecimal string in the `result` field wraps the runtime metadata in 
[SCALE](../advanced/codec) format.

The hex blob starts with a hard-coded magic number `6d657461`, which represents "meta" in plain 
text. The next piece of data shows the version number of the metadata, here we are using `04` to 
represent version 4. We already mentioned that runtime metadata is composed of data from the 
runtime's pallets. In our case we have 9 pallets. After shifting 9 in binary representation two 
bits to the left, we get `24` in hex to represent the length of the array.

The remaining blob encodes the metadata of each pallet. To learn more about decoding different 
types in the [struct](https://crates.parity.io/frame_metadata/struct.ModuleMetadata.html), please 
refer to the [reference docs](https://crates.parity.io/frame_metadata/index.html)..

After decoding the hex blob successfully, you should be able to see similar metadata in the above 
example.

## Format

### Encoded Format

Encoded, runtime metadata is a tuple struct, `RuntimeMetadataPrefixed(u32, RuntimeMetadata)` where 
`u32` is a 4-byte prefix and `RuntimeMetadata` is the encoded metadata enum. Since enums in SCALE 
are encoded by indexing, the first byte (5th byte in the SCALE-encoded byte-string) is the index of 
the enum variant, but also conveniently corresponds to the version of the metadata. 

### Decoded Format

```json
[
  1635018093,
  {
    "V11": {
        "modules": [
            {
                ...
            },
            {
                ...
            }
       ],
       "extrinsic" {
           "version": 4,
           "signed_extensions": [
               "RestrictFunctionality",
               "CheckVersion",
               "CheckGenesis",
               "CheckEra",
               "CheckNonce",
               "CheckWeight",
               "ChargeTransactionPayment",
               "LimitParathreadCommits"
           ]
       }
    }
  }
]
```

The number at the beginning is the prefix, which is the ascii-text "meta" in big-endian. The rest 
of the metadata has two sections: "modules" and "extrinsic". The modules section contains the bulk 
of the information about the runtime's pallets, while the extrinsic sections denotes the version of 
extrinsics in use. Different extrinsic versions may have different formats, especially when 
considering [signed extrinsics](../learn-substrate/extrinsics).

#### Modules

```json
{
    "name": "System",
    "storage": {
      ..
    },
    "calls": [
      ..
    ],
    "event": [
      ..
    ],
    "constants": [
        ..
    ],
    "errors": [
        ..
    ]
}
```

Every module contains the name of the pallet being represented, as well as a "storage" object, 
"calls" array, "event" array, and "errors" array. If "calls" or "event" are empty, they will be a 
`null` value in JSON, if constants or errors are empty, they will be an empty array.

##### Storage Item

```json
{
 "name": "System",
 "storage": {
   "prefix": "System",
   "entries": [
     {
       "name": "Account",
       "modifier": "Default",
       "ty": {
         "Map": {
           "hasher": "Blake2_128Concat",
           "key": "T::AccountId",
           "value": "AccountInfo<T::Index, T::AccountData>",
           "unused": false
         }
       },
       "default": [0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0, 0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0]
       "documentation": [
         " The full account information for a particular account ID."
       ]
    },
    {
        "name": "ExtrinsicCount",
      ..
    },
    {
      "name": "AlLExtrinsicsWeight",
      ..
    }
  ]
},
```

Every storage item defined in a pallet will have a corresponding metadata entry. For example, the 
`Account` item is generated from this in `frame-system`:

```rust
decl_storage! {
    trait Store for Module<T: Trait> as System {
        /// The full account information for a particlar account ID.
        pub Account get(fn account):
            map hasher(blake2_128_concat) T::AccountId => AccountInfo<T::Index, T::AccountData>;
    }
}
```

With the metadata, this lets someone know the types and information required to query the RPC 
storage function to get account information for a specific `AccountId`, assuming they know the type 
that the generic `T::AccountId` resolves to. By combining the `prefix` and the `name` of storage 
and querying the `state_subscribeStorage` RPC endpoint, you could subscribe to new entries in 
`Account` storage.

##### Calls

The metadata will take the information from `decl_module!` about runtime calls. For example, this 
comes from the Timestamp pallet.

```rust
decl_module! {
	pub struct Module<T: Trait> for enum Call where origin: T::Origin {

		/// Set the current time.
		///
		/// This call should be invoked exactly once per block. It will panic at the finalization
		/// phase, if this call hasn't been invoked by that time.
		///
		/// The timestamp should be greater than the previous one by the amount specified by
		/// `MinimumPeriod`.
		///
		/// The dispatch origin for this call must be `Inherent`.
		#[weight = SimpleDispatchInfo::FixedOperational(10_000)]
		fn set(origin, #[compact] now: T::Moment) {
			// ... snip
		}

	}
}
```

The metadata will include:

- `name`: Name of the function in the module.
- `args`: Arguments in function definition. Includes the name and type of each argument.
- `Documentation`: Documentation of the function.

```json
"calls": [
  {
    "name": "set",
    "arguments": [
        {
          "name": "now",
          "ty": "Compact<T::Moment>"
        }
    ],
    "documentation": [
      " Set the current time.",
      "",
      " This call should be invoked exactly once per block. It will panic at the finalization",
      " phase, if this call hasn't been invoked by that time.",
      "",
      " The timestamp should be greater than the previous one by the amount specified by",
      " `MinimumPeriod`.",
      "",
      " The dispatch origin for this call must be `Inherent`."
    ]
  }
],
```

##### Events

This metadata snippet is generated from this declaration in `frame-system`:

```Rust
decl_event!(
    /// Event for the System module
    pub enum Event<T> where AccountId = <T as Trait>::AccountId {
        /// An extrinsic completed successfully.
        ExtrinsicSuccess(DispatchInfo),
        /// An extrinsic failed.
        ExtrinsicFailed(DispatchError, DispatchInfo),
    }
)
```

The metadata will include event information:

```json
"event": [
    {
      "name": "ExtrinsicSuccess",
      "arguments": [
        "DispatchInfo"
      ],
      "documentation": [
          " An extrinsic completed successfully."
      ]
    },
    {
      "name": "ExtrinsicFailed",
      "arguments": [
        "DispatchError",
        "DispatchInfo"
      ],
      "documentation": [
        " An extrinsic failed."
      ]
    },
],
```

##### Constants

The metadata will include any module constants. From `pallet-babe`:

```rust
decl_module! {
    /// The BABE Pallet        
	pub struct Module<T: Trait> for enum Call where origin: T::Origin {
        /// The number of **slots** that an epoch takes. We couple sessions to
        /// epochs, i.e. we start a new session once the new epoch begins.
        const EpochDuration: u64 = T::EpochDuration::get();
        
        // ...
    }
}
```

This module will produce the following in the metadata:

```json
"constants": [
    {
      "name": "EpochDuration",
      "ty": "u64",
      "value": [
        88,
        2,
        0,
        0,
        0,
        0,
        0,
        0
      ],
      "documentation": [
        " The number of **slots** that an epoch takes. We couple sessions to",
        " epochs, i.e. we start a new session once the new epoch begins."
      ]
    },
],
```

The metadata also includes constants defined in the runtime's `lib.rs`. For example, from Kusama:

```rust
parameter_types! {
    pub const EpochDuration: u64 = EPOCH_DURATION_IN_BLOCKS as u64;
}
```

Where `EPOCH_DURATION_IN_BLOCKS` is a constant defined in `runtime/src/constants.rs`.

##### Errors

Metadata will pull all the possible runtime errors from `decl_error!`. For example, from 
`frame-system`:

```Rust
decl_error! {
    /// Error for the system module
    pub enum Error for Module<T: Trait> {
        /// The name of specification does not match between the current runtime
        /// and the new runtime.
        InvalidSpecName,
    }
}
```

This will generate the following metadata:

```json
"errors": [
    {
        "name": "InvalidSpecName",
        "documentation": [
            " The name of specification does not match between the current runtime",
            " and the new runtime."
         ]
    },
]
```

These are errors that could occur during the submission of an extrinsic. In this case, the `InvalidSpecName` error could be raised from `frame-system`.

## Next Steps

### Learn More

- [Storage](storage)
- [SCALE](../advanced/codec)
- [Macros](macros)
- [Events](events)
- [Extrinsics](../learn-substrate/extrinsics)

<!--
### Examples

-
-->

### References

- [Metadata](https://crates.parity.io/frame_metadata/index.html)
