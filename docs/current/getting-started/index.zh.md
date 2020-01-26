---
slug: index
lang: zh
title: 概貌
---

## Prerequisites

To develop in the Substrate ecosystem, you must set up your developer environment. Depending on your operating system, these instructions may be different.

### Debian

Run:

```bash
sudo apt install -y cmake pkg-config libssl-dev git gcc build-essential clang libclang-dev
```

### macOS

Install the [Homebrew package manager](https://brew.sh/), then run:

```bash
brew install openssl cmake llvm
```

## Install

Install Substrate and all it's required dependencies with a single command. (Be patient, this can take up to 30 minutes)

```bash
curl https://getsubstrate.io -sSf | bash -s -- --fast
```