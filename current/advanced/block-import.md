---
slug: block-import
lang: en
title: The Block Import Pipeline
---


## The Import Queue

The import queue is an abstract worker queue that takes in blocks and imports them in order

In order to do it, it uses the import pipeline through the `BlockImport` trait

Adding to that, the import queue is the stage at which we'd do parallel pre-verification, whereas block import is synchronous. We don't do any parallelization now but it would look different for each engine, hence the import queue is provided by each engine



Answer to "what's the difference between finality proof and justification"

grandpa commit: a message sent at the end of each round that contains all the precommits in the round necessary to finalize a block (https://github.com/paritytech/finality-grandpa/blob/master/src/lib.rs#L271)
justification: a grandpa commit + set of all block headers referenced by the precommits (https://github.com/paritytech/substrate/blob/master/client/finality-grandpa/src/justification.rs#L40). having all votes ancestries blocks is necessary in order to prccess them, so this message is like a standalone proof that a given block was finalized
so the above only allows proving whatever was finalized in GRANDPA (i.e. the commit states that a given block was finalized in the round), and GRANDPA doesn't necessarily finalize each block at a time. e.g. for example on round 1 we might finalize block #10 and on round 2 we might finalize block #100, which means we won't have any justifications for the blocks in between. #99 is finalized implicitly because #100 was finalized.
so this is why the finality proof exists, it allows you to prove finality of a block B using a proof of block B' (where B' > B)

finality proof: justification for B', headers [B, B'], proof of authorities call at B' (https://github.com/paritytech/substrate/blob/master/client/finality-grandpa/src/finality_proof.rs)
(edited)
1+R
11:53
the finality proofs are only used by the light client, but slava mentioned the other day that maybe they can be dropped if the light client uses the same import pipeline as the full node
