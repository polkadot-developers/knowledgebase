---
slug: weight
lang: en
title: Transaction Weight
---

Resources available to chains are limited. The resources include memory usage, storage I/O,
computation, transaction/block size and state database size. There are several mechanisms to manage
access to resources and to prevent individual components of the chain from consuming too much of any
resource. Weights are the mechanism used to manage the _time is takes to validate_ a block.
Generally speaking, this comes from limiting the storage I/O and computation.

NOTE: Weights are not used to restrict access to other resources, such as storage itself or memory
footprint. Other mechanisms must be used for this.

The amount of weight a block may contain is limited, and optional weight consumption (i.e. weight
that is not required to be deployed as part of the block's initialization or finalization phases nor
used in mandatory inherent extrinsics) will generally be limited through economic measures --- or in
simple terms, through transaction fees. The fee implications of the weight system are covered in the
[Transaction Fees document](../runtime/fees).

## Weight Fundamentals

Weights represent the _limited_ time that your blockchain has to validate a block. This includes
computational cycles, and storage I/O. A custom implementation may use complex structures to express
this. Substrate weights are simply a
[numeric value](https://substrate.dev/rustdocs/master/frame_support/weights/type.Weight.html).

A weight calculation should always:

- Be computable **ahead of dispatch**. A block producer should be able to examine the weight of a
  dispatchable before actually deciding to accept it or not.
- Consume few resources itself. It does not make sense to consume similar resources computing a
  transaction's weight as would be spent to execute it. Thus, weight computation should be much
  lighter than dispatch.
- Be able to determine resources used without consulting on-chain state. Weights are good at
  representing _fixed_ measurements or measurements based solely on the parameters of the
  dispatchable function where no expensive I/O is necessary. Weights are not so useful when the cost
  is dependent on the chain-state.

In the case that the weight of a dispatchable is heavily dependent on chain-state case, two options
are available:

- Determine or introduce a forced upper limit to the amount of weight a dispatchable could possibly
  take. If the difference between the enforced upper limit and the least possible amount of weight a
  dispatchable could take is small, then it can just be assumed to always be at the upper limit of
  the weight without consulting the state. If the difference is too great, however, then the
  economic cost of making lesser transactions might be too great which will warp the incentives and
  create inefficiencies in throughput.
- Require the effective weight (or precursors that can be used to efficiently compute it) be passed
  in as parameters to the dispatch. The weight charged should be based on these parameters but also
  cover the amount of time it takes to verify them during dispatch. Verification must take place to
  ensure the weighing parameters correspond accurately to on-chain state and if they don't then the
  operation should gracefully error.

The [System pallet](https://substrate.dev/rustdocs/master/frame_system/struct.Module.html) is
responsible for accumulating the weight of each block as it gets executed and making sure that it
does not exceed the limit. The
[Transaction Payment pallet](https://substrate.dev/rustdocs/master/pallet_transaction_payment/index.html)
is responsible for interpreting these weights and deducting fees based upon them. The weighing
function is part of the runtime so it can be upgraded if needed.

## Block Weight and Length Limit

Aside from affecting fees, the main purpose of the weight system is to prevent a block from being
filled with transactions that would take too long to execute. While processing transactions within a
block, the System pallet accumulates both the total length of the block (sum of encoded transactions
in bytes) and the total weight of the block. If either of these numbers surpass the limits, no
further transactions are accepted in that block. These limits are defined in
[`MaximumBlockLength`](https://substrate.dev/rustdocs/master/frame_system/trait.Trait.html#associatedtype.MaximumBlockLength)
and
[`MaximumBlockWeight`](https://substrate.dev/rustdocs/master/frame_system/trait.Trait.html#associatedtype.MaximumBlockWeight).

One important note about these limits is that a portion of them are reserved for the `Operational`
dispatch class. This rule applies to both of the limits and the ratio can be found in
[`AvailableBlockRatio`](https://substrate.dev/rustdocs/master/frame_system/trait.Trait.html#associatedtype.AvailableBlockRatio).

For example, if the block length limit is 1 megabyte and the ratio is set to 80%, all transactions
can fill the first 800 kilobytes of the block while the last 200 can only be filled by the
operational class.

There is also a `Mandatory` dispatch class that can be used to ensure an extrinsic is always
included in a block regardless of its impact on block weight. Please refer to the
[Transaction Fees document](../runtime/fees) to learn more about the different dispatch classes and
when to use them.

## Next Steps

### Learn More

- Substrate Recipes contains examples of both
  [custom weights](https://github.com/substrate-developer-hub/recipes/tree/master/pallets/weights)
  and custom
  [WeightToFee](https://github.com/substrate-developer-hub/recipes/tree/master/runtimes/weight-fee-runtime).
- The [Example](https://github.com/paritytech/substrate/blob/master/frame/example/src/lib.rs)
  pallet.

### Examples

- See an example of
  [adding a transaction weight](https://substrate.dev/recipes/3-entrees/weights.html) to a custom
  runtime function.

### References

- [Transaction Payment pallet](https://github.com/paritytech/substrate/blob/master/frame/transaction-payment/src/lib.rs)
- [Weights](https://github.com/paritytech/substrate/blob/master/frame/support/src/weights.rs)
