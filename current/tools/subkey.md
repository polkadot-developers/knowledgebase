---
slug: subkey
lang: en
title: The subkey Tool
---

`subkey` is a key-generation utility that is developed alongside Substrate. Its main features are generating [sr25519](TK) and [ed25519]() key pairs, encoding SS58 addresses, and restoring keys from mnemonics and raw seeds. It can also create and verify signatures, including for encoded transactions.

## Installation

### Build from Source

The Subkey binary, `subkey`, is also installed along with the [Substrate installation](TK). If you want to play with just Subkey (and not Substrate), you will need to have the Substrate dependencies. Use the following two commands to install the dependencies and Subkey, respectively:

```bash
$ curl https://getsubstrate.io -sSf | bash -s -- --fast
$ cargo install --force --git https://github.com/paritytech/substrate subkey
```

### Compiling with Cargo

If you already have the [Substrate repository](https://github.com/paritytech/substrate), you can build Subkey with:

```bash
$ cargo build -p subkey --release
```

This will generate the `subkey` binary in `./target/release/subkey`.

## Generating Keys

Generate an sr25519 key by running:

```bash
$ subkey generate

Secret phrase `spend report solution aspect tilt omit market cancel what type cave author` is account:
  Secret seed:      0x554b6fc625fbea8f56eb56262d92ccb083fd6eaaf5ee9a966eaab4db2062f4d0
  Public key (hex): 0x143fa4ecea108937a2324d36ee4cbce3c6f3a08b0499b276cd7adb7a7631a559
  Account ID:       0x143fa4ecea108937a2324d36ee4cbce3c6f3a08b0499b276cd7adb7a7631a559
  SS58 Address:     5CXFinBHRrArHzmC6iYVHSSgY1wMQEdL2AiL6RmSEsFvWezd
```

For even more security, use `--words 24` (supports 12, 15, 18, 21, and 24):

```bash
$ target/release/subkey generate --words 24
Secret phrase `enroll toss harvest pilot size end skin clog city knock bar cousin mirror journey coil used eye describe puzzle govern soup sort second cattle` is account:
  Secret seed:      0x7590d644baa64600735ab927b6c353b9594a2cf42fe4d57c2d0e639615b37a6a
  Public key (hex): 0xc61c17f9a3d48ff3f0700081ac7a9c92ef05758d5cc069c41247e4392eb71e00
  Account ID:       0xc61c17f9a3d48ff3f0700081ac7a9c92ef05758d5cc069c41247e4392eb71e00
  SS58 Address:     5GYTgcrom3pxWyHCcPPZX6iBB8xWJPARgLAng7MK4ZFaBAza
```

Subkey encodes the address depending on the network. You can use the `-n` or `--network` flag to change this. See `subkey help` for supported networks.

To generate an ed25519 key, pass the `-e` or `--ed25519` flag:

```bash
$ subkey -e generate
Secret phrase `security snow crunch treat tone skill develop nominee hat slim omit stool` is account:
  Secret seed:      0xad282c9eda80640f588f812081d98b9a2333435f76ba4ad6258e9c6f4a488363
  Public key (hex): 0xf6a7ac42a6e1b5329bdb4e63c8bbafa5301add8102843bfe950907bd3969d944
  Account ID:       0xf6a7ac42a6e1b5329bdb4e63c8bbafa5301add8102843bfe950907bd3969d944
  SS58 Address:     5He7SpmVsdhoEKC5uwvPDngoCXECCeh8VrxoxnQTh1mgPiZa
```

The output gives us the following information about our key:

- **Secret phrase** (aka "mnemonic phrase") - A series of English words that encodes the seed in a more human-friendly way. Mnemonic phrases were first introduced in Bitcoin (see [BIP39](https://github.com/bitcoin/bips/blob/master/bip-0039.mediawiki)) and make it much easier to write down your key by hand.
- **Secret Seed** (aka "Private Key" or "Raw Seed") - The minimum necessary information to restore the key pair. All other information is calculated from the seed.
- **Public Key** (aka "**Account ID**") - The public half of the cryptographic key pair in hexadecimal.
- **SS58 Address** (aka "Public Address") - An SS58-encoded address based on the public key.

Currently we support a few cryptographies:

- `-e`, `--ed25519`: Ed25519/BIP39 cryptography
- `-k`, `--secp256k1`: SECP256k1/ECDSA/BIP39 cryptography
- `-s`, `--sr25519`: Schnorr/Ristretto x25519/BIP39 cryptography

You can also create a vanity address, although you will not receive a mnemonic seed:

```bash
$ subkey vanity joe  --number 1
Generating key containing pattern 'joe'
100000 keys searched; best is 185/189 complete
200000 keys searched; best is 187/189 complete
300000 keys searched; best is 187/189 complete
best: 189 == top: 189
Secret Key URI `0x380ba96e07933b89c2b3b6d3fa98b695aab99070b8982817b74044c6fa6a375b` is account:
  Secret seed:      0x380ba96e07933b89c2b3b6d3fa98b695aab99070b8982817b74044c6fa6a375b
  Public key (hex): 0x4a0e48c38ae880f7eee86c8a75c6391ecb2de35adf49ee30cd1631e82de1b001
  Account ID:       0x4a0e48c38ae880f7eee86c8a75c6391ecb2de35adf49ee30cd1631e82de1b001
  SS58 Address:     
```

Notice the SS58 Address `5D**joe**JSa7m4EsuSKwDN1GkrMtTiWwSMGxESQ6KSNPETPaPna` contains the string `joe`. 

`--number` is for legacy purpose, and so it is going to generate one vanity address only no matter what value you pass in.
