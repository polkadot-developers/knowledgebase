---
slug: governance
lang: en
title: Governance Modules
---

Fundamentally, blockchains are data structures that enable decentralized consensus - they are platforms that allow people
to build trustless systems for making complex decisions. Blockchains and the novel cryptographic protocols on which they
are built have created new mechanisms for human interaction, which implies that they must expose capabilities to allow
_people_ to interact with one another. It may seem alluring to enshrine blockchain's characteristics of immutability and
determinism into an inflexible system that is incapable of change, but only the simplest of such systems can be sustainable
in the long-term. As people use blockchain platforms to solve increasingly complex problems, the need to create dynamic
systems that are designed for change will also increase. Some of these changes will be required to solve entirely new
problems that occur in decentralized settings. However, even familiar problems will require novel solutions, since
people participate in blockchain networks in order to create systems that are more fair and equitable and require less
trust than traditional ones. The term "governance" refers to the set of systems and processes that people create to
manage decision-making and change within a community. The creators of Substrate have designed a rich set of flexible
modules that expose a range of capabilities related to network governance. The purpose of this document is to discuss
the design decisions that were taken into consideration when creating these modules, explain the capabilities that they
expose and provide helpful context and information for using them in concert to build the governance systems that best
suit your blockchain.

# Governance Modules

Substrate's governance modules have been designed according to the same principle as the Substrate framework itself -
they are intended to encapsulate composable building blocks that can be used to express increasingly complex systems.
Some of the Substrate governance modules, such as the Sudo module, expose simple interfaces that are straightforward and
relatively unopinionated; others, such as the Democracy modules, define opinionated APIs that encapsulate more complex
governance concepts.

## Collective

Collectives are groups that form around shared values and ideas. The Collective module exposes capabilities related to
managing the members of a collective as well as features that allow collectives to make decisions and then act on those
decisions.

## Democracy

## Elections

## Phragmen Elections

## Membership

## Sudo

## Treasury
