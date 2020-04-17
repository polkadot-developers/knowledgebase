# Metadata V11 (written 04/07-09/2020)
### All Metadata examples taken from block 1768321 on the Kusama Chain CC3
### Full Metadata Here: https://gist.github.com/insipx/db5e49c0160b1f1bd421a3c34fefdf48

## Purpose
The Substrate Metadata lets you inspect type/error information of Calls, Events, Storage and Constants of every pallet in Substrate, as well as any pallet that is created using substrate macros. It's automatically generated within substrate macros (decl_module!, decl_storage!, decl_event!, decl_error!). With the metadata, you are able to acquire the information necessary to query storage and code around a specific runtime's types without importing the code from that runtime, or any previous runtime. It's a faster way of acquiring necessary information needed to query parts of a chain without the compilation overhead. Metadata for a specific version of the chain is stored on a per-block basis. So querying the metadata for a blocks hash a few months back (with an archive node, for example), could possibly result in acquiring an older version of metadata. Metadata will include information specific to a singular runtime rather than being generic over any runtime.


## How to Use
### Rust
The easiest way to get the metadata is by querying the automatically generated JSONRPC function `state_getMetadata` (client/rpc-api/src/state/mod.rs in substrate repository). This will return a Vector of SCALE-encoded bytes (`Bytes` where `Bytes` is a newstruct wrapping a `Vec<u8>`). If using rust you can decode into the `RuntimeMetadataPrefixed` struct using substrate 'frame-metadata' and 'parity-scale-codec' libraries. Older versions of metadata would need to be decoded into their corresponding versioned `RuntimeMetadataPrefixed`. Some helpful libraries, `substrate-subxt` fetch the metadata and decodes it into the latest `RuntimeMetadataPrefixed` struct and converts it into a helpful format for you, while `desub` doesn't fetch metadata from RPC, it will decode and convert historical metadata from version 8. Effort is underway to combine desub/subxt. Once decoded into `RuntimeMetadataPrefixed` the struct may be serialized into JSON with serde. If you'd prefer to use the RPC more directly, the (JSONRPC)[https://github.com/paritytech/jsonrpc] or (jsonrpsee)[https://github.com/paritytech/jsonrpsee] libraries provide an interface to do so from rust, and both are created with substrate/polkadot in mind.

### Javascript

If you are using **Javascript**,  [polkadot-js/api](https://polkadot.js.org/api/) already provides friendly APIs to interact with Substrate blockchain, including the [getMetadata](https://polkadot.js.org/api/METHODS_RPC.html#json-rpc) function.
You can try the following code snippets to fetch the metadata in this [Substrate UI](https://polkadot.js.org/apps/#/js) page:

```javascript
const { magicNumber, metadata } = await api.rpc.state.getMetadata();

console.log( 'Magic number: ' + magicNumber );
console.log( 'Metadata: ' + metadata.raw );
```

### Other Languages
you can send a **WebSocket message** or **HTTP POST request** to a Substrate node endpoint by using any existing client. The message or body for `getMetadata` is:

```json
{
  "id": 1,
  "jsonrpc": "2.0",
  "method": "state_getMetadata",
  "params": []
}
```
you can leave 'params' empty, or if you want to fetch the metadata for a specific block:

```json
{
  "id": 1,
  "jsonrpc": "2.0",
  "method": "state_getMetadata",
  "params": ["ca15c2f1e1540517697b6b5f2cc6bc0c60876a1a1af604269b7215970798bbed"]
}
```
where `0xca15c2f1e1540517697b6b5f2cc6bc0c60876a1a1af604269b7215970798bbed` is the hash of block 1768321


The response looks like following contents, but with much more information in `result` field:

```json
{
    "jsonrpc": "2.0",
    "result": "0x6d65746104241873797374656d1853797374656d012c304163636f756e744e6f6e636501010130543a3a4163636f756e74...",
    "id": 1
}
```

The hexadecimal string in the `result` field wraps the runtime metadata in SCALE format. Go to [Low-level Data Format](overview/low-level-data-format.md) page to learn how to decode different types and get more information about the specification and implementations.

Our hex blob starts with a hard coded magic number `6d657461` which represents *meta* in plain text. The next piece of data shows the version number of the metadata, here we are using `04` to represent version 4. We already mentioned that runtime metadata is composed by available pallets. In our case we have 9 pallets. After shifting 9 in binary representation two bits to the left, we get `24` in hex to represent the length of the array.

The remaining blob encodes the metadata of each pallet. Learning more about decoding different types in the [struct](https://substrate.dev/rustdocs/master/frame_metadata/struct.ModuleMetadata.html), please refer to the [reference docs](https://substrate.dev/rustdocs/master/frame_metadata/index.html) and [Low-level Data Format](overview/low-level-data-format.md) page.

After decoding the hex blob successfully, you should be able to see similar metadata in the above example.

For Javascript/other langs this part should be the same as described in https://github.com/substrate-developer-hub/substrate-developer-hub.github.io/blob/4fb25004b22d0a72d32cde7d763c0cdf752334d4/docs/runtime/runtime-metadata.md (I think)

## Format
### Encoded Format
Encoded, runtime metadata is a tuple struct, `RuntimeMetadataPrefixed(u32, RuntimeMetadata)` where 'u32' is a 4-byte prefix, and RuntimeMetadata is the Encoded Metadata Enum. Since enums in SCALE are encoded by indexing, the first byte (5th byte in the SCALE-encoded byte-string) is the index of the enum variant, but also conveniently corresponds to the version of the Metadata. 

### Decoded Format
### JSON
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

the number at the beginning is the 'prefix', which is the ascii-text 'meta' in big-endian 
That's the general outline of the metadata (with the bulk of it, the actual 'modules' portion, which corresponds to substrate pallets, taken out). the "extrinsic" section denotes the version of extrinsic being used. Different extrinsic versions may have different formats, especially when considering signed extrinsics. Signed Extensions denotes which extra information is added to signed extrinsics in this version of the runtime.

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

Every module contains the name of the pallet being represented, as well as a "storage" object, "calls" array, "event" array, and "errors" array. If "calls" or "event" are empty, they will be a 'null' value in JSON, if constants or errors are empty, they will be an empty array. 


##### Storage Item:
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

Every storage item defined in a pallet will have a corresponding metadata entry. For example, the "Account" item is generated from this in "frame/system/lib.rs" lino 340:
```rust
decl_storage! {
    trait Store for Module<T: Trait> as System {
        /// The full account information for a particlar account ID
        pub Account get(fn account):
            map hasher(blake2_128_concat) T::AccountId => AccountInfo<T::Index, T::AccountData>;
    }
}
```
With the metadata, this lets someone know the types and information required to query the RPC storage function to get account information for a specific AccountId, assuming they know the type that the generic "T::AccountId" resolves to. By combining the `"prefix"` and the `"name"` of storage and querying the `"state_subscribeStorage"` you could subscribe to new entries in `Account` storage. Or you could query a specific account combining the prefix + name + AccountId and hashing with blake2_128_concat. (Not sure if this is still entirely how this works with the storage changes). But point is, with the information in the metadata, you will have all you need to query storage for any pallet. The values are pretty much the same as in the original doc, except for some name changes (IE fallback -> default)

##### Calls:

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

This is generated from this piece of declarational code:
```rust
decl_module! {
	pub struct Module<T: Trait> for enum Call where origin: T::Origin {
        // ... (redacted)

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
			// ... (redacted)
		}

	    // ... (redacted)
	}
}
```
- `name`: name of the function in the module
- `args`: Arguments in function definition. Includes the name of the argument and type.
- `Documentation`: Documentation of the function

If the type `T::Moment` is known, then an application of this metadata could be decoding extrinsics found in a block. Timestamps extrinsics/inherents are unsigned so there's no need to decode a signature

##### Events:
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

This metadata snippet is generated from this declaration in `frame/system/src/lib.rs`

```rust
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

##### Constants:

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

generated from `frame/babe/src/lib.rs`:

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
and in `polkadot/runtime/kusama/src/lib.rs`:

```rust
parameter_types! {
    pub const EpochDuration: u64 = EPOCH_DURATION_IN_BLOCKS as u64;
}
```
where `EPOCH_DURATION_IN_BLOCKS` is a constant defined in `polkadot/runtime/kusama/src/constants.rs`

Constants just constants, they won't change for the duration of the runtime that the metadata is relevant for. Therefore, they are directly represented in the metadata with a `value`, `type`, and `name`

##### Errors
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

generated from `frame/system/src/lib.rs`:

```rust
decl_error! {
    /// Error for the system module
    pub enum Error for Module<T: Trait> {
        /// the name of specification does not match between the current runtime
        /// and the new runtime.
        InvalidSpecName,
        
        /// ... (redacted)
    }
}
```
These are errors that could occur during the submission of an extrinsic. In this case, the `InvalidSpecName` error could be raised during the `System` `set_code` extrinsic. This will then raise an `ExtrinsicFailed` event, with the actual error for the module contained within the `DispatchError` enum data type in `sp-runtime`. 


