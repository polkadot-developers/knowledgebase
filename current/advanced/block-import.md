---
slug: block-import
lang: en
title: The Block Import Pipeline
---


## The Import Queue

The import queue is an abstract worker queue present in every Substrate node. It is not part of the runtime. The Import Queue is responsible for processing pieces of incoming information, verifying them, and if they are valid, importing them into the node's state. The most fundamental piece of information that the import queue processes is blocks themselves, but it is also responsible for importing consensus-related messages such as Justifications and, in light clients, finality proofs.

The Import queue collects incoming blocks from the network or from other parts of the local node, and stores them in a pool. The blocks are later checked for validity and discarded if they are not valid. Finally, the block is imported when the import queue is instructed to import it by the consensus engine.

The import queue is codified abstractly in Substrate by means of the [`ImportQueue` trait](https://substrate.dev/rustdocs/v2.0.0-alpha.8/sp_consensus/import_queue/trait.ImportQueue.html). The use of a trait allows each consensus engine to provide its own specialized implementation of the Import Queue which may take advantage of optimization opportunities such as verifying multiple blocks in parallel as they come in across the network.

## The Basic Queue
Substrate also provides a default implementation known as the [`BasicQueue`](https://substrate.dev/rustdocs/v2.0.0-alpha.8/sp_consensus/import_queue/struct.BasicQueue.html). The `BasicQueue` does not do any kind of optimization, rather it performs the verification and import steps sequentially. It does, however, abstract the notion of verification through the use of the [`Verifier`](https://substrate.dev/rustdocs/v2.0.0-alpha.8/sp_consensus/import_queue/trait.Verifier.html) trait.

Any consensus engine that relies on the `BasicQueue` must implement the `Verifier` trait. In the simplest cases, such as Sunstrate's instant seal consensus, the `Verifier` is simple declares all blocks as valid. In slot-based engines check that the blocks timestamp actually falls in the correct time window for the slot it is claiming.

## The Block Import Trait

When the import queue is instructed to import a block, it passes the block in question to a method provided by the [`BlockImport` trait](https://substrate.dev/rustdocs/v2.0.0-alpha.8/sp_consensus/block_import/trait.BlockImport.html). This `Block Import` trait provides the behavior of importing a block into the node's local state.

One implementor of the `BlockImport` trait that is used in every Substrate node is the [`Client`](https://substrate.dev/rustdocs/v2.0.0-alpha.8/sc_service/client/index.html) which contains the node's entire block database. When a block is imported into the client it is added to the main database of blocks that the node knows about.

## The Block Import Pipeline

In the simplest cases, such as manual-seal nodes, blocks are imported directly into the client. But most consensus engines will need to perform additional verification on incoming blocks, update their own local state databases, or both. To allow consensus engines this opportunity, it is common to wrap the client in another struct which is also `BlockImport`. This nesting leads to the term "block import pipeline".

An example of this wrapping is the [`PowBlockImport`](https://substrate.dev/rustdocs/v2.0.0-alpha.8/sc_consensus_pow/struct.PowBlockImport.html) which holds a reference to another type that is `BlockImport`. This allows the PoW consensus engine to do it's own import-related bookkeeping and then pass the block to the nested `BlockImport`, probably the client. This pattern is also demonstrated in the `AuraBlockImport`, `BabeBlockImport`, and `GrandpaBlockImport`.

This nesting need not be limited to one level. In fact it is common for nodes that use both an authoring engine and a finality gadget to layer the nesting even more deeply. For example Polkadot's block import pipeline consists of a `GrandpaBlockImport` which wraps a `BabeBlokImport` which, itself, wraps the `Client`.

## Learn More

Several of the Recipes' nodes demonstrate the block import pipeline:

* [Manual Seal](https://substrate.dev/recipes/3-entrees/manual-seal.html) - all blocks are valid so the block import pipeline is just the client
* [Basic PoW](https://substrate.dev/recipes/3-entrees/basic-pow.html) - the import pipeline includes PoW and the client
* [Hybrid Consensus](https://substrate.dev/recipes/3-entrees/hybrid-consensus.html) - the import pipeline is PoW, then Grandpa, then the client