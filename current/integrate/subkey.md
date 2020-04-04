---
slug: subkey
lang: en
title: The subkey Tool
---

`subkey` is a key-generation utility that is developed alongside Substrate. Its main features are generating key pairs (currently supporting [Sr25519](https://wiki.polkadot.network/docs/en/learn-cryptography), [Ed25519](https://en.wikipedia.org/wiki/EdDSA#Ed25519), and [SECP256k1](https://en.bitcoin.it/wiki/Secp256k1) cryptographies), encoding SS58 addresses, and restoring keys from mnemonics and raw seeds. It can also create and verify signatures, including for encoded transactions.

## Installation

### Build from Source

The Subkey binary, `subkey`, is also installed along with the [Substrate installation](https://substrate.dev/docs/en/overview/getting-started#unix-based-operating-systems). If you want to play with just Subkey (and not Substrate), you will need to have the Substrate dependencies. Use the following two commands to install the dependencies and Subkey, respectively:

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

Currently `subkey` support the following cryptographies (key types):

- `-s`, `--sr25519`: Schnorr/Ristretto x25519/BIP39 cryptography
- `-e`, `--ed25519`: Ed25519/BIP39 cryptography
- `-k`, `--secp256k1`: SECP256k1/ECDSA/BIP39 cryptography

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
  SS58 Address:     5DjoeJSa7m4EsuSKwDN1GkrMtTiWwSMGxESQ6KSNPETPaPna
```

Notice the SS58 Address `5D**joe**JSa7m4EsuSKwDN1GkrMtTiWwSMGxESQ6KSNPETPaPna` contains the string `joe`. 

`--number` is for legacy purpose, so it is going to generate one vanity address no matter what value is passed in.

## Password Protected Keys

To generate a key that is password protected, use the `-p <password>` or `--password <password>` flag:

```bash
$ subkey --password "pencil laptop kitchen cutter" generate
Secret phrase `image stomach entry drink rice hen abstract moment nature broken gadget flash` is account:
  Secret seed:      0x11353649ecfb97ed44aa4b3516c646fa49ecd3f71cc0445647a25379f848b336
  Public key (hex): 0x1641450068c6ed825726c2a854d830352b75fb1a05e6fbcc9bc47a398a581a29
  Account ID:       0x1641450068c6ed825726c2a854d830352b75fb1a05e6fbcc9bc47a398a581a29
  SS58 Address:     5CZtJLXtVzrBJq1fMWfywDa6XuRwXekGdShPR4b8i9GWSbzB
```

Note that the "Secret seed" is _not_ password protected and can still recover the account. The "Secret phrase," however, is not sufficient to recover the account without the password.

## Inspecting Keys

The inspect command recalculates a key pair's public key and public address given its seed. This shows that it is sufficient to back up the seed alone.

```bash
$ subkey inspect 0x380ba96e07933b89c2b3b6d3fa98b695aab99070b8982817b74044c6fa6a375b
Secret Key URI `0x380ba96e07933b89c2b3b6d3fa98b695aab99070b8982817b74044c6fa6a375b` is account:
  Secret seed:      0x380ba96e07933b89c2b3b6d3fa98b695aab99070b8982817b74044c6fa6a375b
  Public key (hex): 0x4a0e48c38ae880f7eee86c8a75c6391ecb2de35adf49ee30cd1631e82de1b001
  Account ID:       0x4a0e48c38ae880f7eee86c8a75c6391ecb2de35adf49ee30cd1631e82de1b001
  SS58 Address:     5DjoeJSa7m4EsuSKwDN1GkrMtTiWwSMGxESQ6KSNPETPaPna
```

You can also inspect the key by its mnemonic phrase.

```bash
$ subkey inspect "spend report solution aspect tilt omit market cancel what type cave author"
Secret phrase `spend report solution aspect tilt omit market cancel what type cave author` is account:
  Secret seed:      0x554b6fc625fbea8f56eb56262d92ccb083fd6eaaf5ee9a966eaab4db2062f4d0
  Public key (hex): 0x143fa4ecea108937a2324d36ee4cbce3c6f3a08b0499b276cd7adb7a7631a559
  Account ID:       0x143fa4ecea108937a2324d36ee4cbce3c6f3a08b0499b276cd7adb7a7631a559
  SS58 Address:     5CXFinBHRrArHzmC6iYVHSSgY1wMQEdL2AiL6RmSEsFvWezd
```

You can inspect **password** protected keys either by passing the `--password` flag or using `///` at the end of the mnemonic:

```bash
$ subkey -p "pencil laptop kitchen cutter" inspect "image stomach entry drink rice hen abstract moment nature broken gadget flash"
Secret phrase `image stomach entry drink rice hen abstract moment nature broken gadget flash` is account:
  Secret seed:      0x11353649ecfb97ed44aa4b3516c646fa49ecd3f71cc0445647a25379f848b336
  Public key (hex): 0x1641450068c6ed825726c2a854d830352b75fb1a05e6fbcc9bc47a398a581a29
  Account ID:       0x1641450068c6ed825726c2a854d830352b75fb1a05e6fbcc9bc47a398a581a29
  SS58 Address:     5CZtJLXtVzrBJq1fMWfywDa6XuRwXekGdShPR4b8i9GWSbzB

$ subkey inspect "image stomach entry drink rice hen abstract moment nature broken gadget flash///pencil laptop kitchen cutter"
Secret Key URI `image stomach entry drink rice hen abstract moment nature broken gadget flash///pencil laptop kitchen cutter` is account:
  Secret seed:      0x11353649ecfb97ed44aa4b3516c646fa49ecd3f71cc0445647a25379f848b336
  Public key (hex): 0x1641450068c6ed825726c2a854d830352b75fb1a05e6fbcc9bc47a398a581a29
  Account ID:       0x1641450068c6ed825726c2a854d830352b75fb1a05e6fbcc9bc47a398a581a29
  SS58 Address:     5CZtJLXtVzrBJq1fMWfywDa6XuRwXekGdShPR4b8i9GWSbzB
```

Let's say you want to use the same private key on Kusama. Use `-n` to get your address formatted for Kusama. Notice that the public key is the same, but the address has a different format.

```bash
$ subkey --network kusama inspect "spend report solution aspect tilt omit market cancel what type cave author"
Secret phrase `spend report solution aspect tilt omit market cancel what type cave author` is account:
  Secret seed:      0x554b6fc625fbea8f56eb56262d92ccb083fd6eaaf5ee9a966eaab4db2062f4d0
  Public key (hex): 0x143fa4ecea108937a2324d36ee4cbce3c6f3a08b0499b276cd7adb7a7631a559
  Account ID:       0x143fa4ecea108937a2324d36ee4cbce3c6f3a08b0499b276cd7adb7a7631a559
  SS58 Address:     D2sP6XA4DBn3eadsRMYBPoggcDbCuSWUYZ5V63PifURFkcm
```

## Inserting Keys to Keystore

You can insert fixed keys into a Substrate-based node keystore with `subkey`.

```
$ subkey insert <SEED> <KEY_TYPE> <node-url>

# Exmaple
$ subkey insert 0x554b6fc625fbea8f56eb56262d92ccb083fd6eaaf5ee9a966eaab4db2062f4d0 test
```

- `<SEED>` - seed or secrete phrase
- `<KEY_TYPE>` - a 4-character long string to specify the key type
- `<node-url>` - default to `http://localhost:9933`

## HD Derivation

Subkey supports hard and soft hierarchical deterministic (HD) key derivation compliant with [BIP32](https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki). HD keys allow you to have a master seed and define a tree with a key pair at each leaf. The tree has a similar structure to a filesystem and can have any depth you please.

Hard and soft key derivation both support:

- `parent private key + path --> child private key`
- `parent private key + path --> child public key`

Further, soft key derivation supports:

- `parent public key + path --> child public key`

### Hard Key Derivation

You can derive a hard key child using `//` after the mnemonic phrase:

```bash
$ subkey inspect "spend report solution aspect tilt omit market cancel what type cave author//joe//polkadot//0"
Secret Key URI `spend report solution aspect tilt omit market cancel what type cave author//joe//polkadot//0` is account:
  Secret seed:      0xc25dda466c7749ac966b95e2ef8f5e758a063d26ac4d1895f479964aaf5679cc
  Public key (hex): 0xb49bdb8f2aa714b6e8b62426d03a1506878fc1f2937bdd396bc2102c95613f4b
  Account ID:       0xb49bdb8f2aa714b6e8b62426d03a1506878fc1f2937bdd396bc2102c95613f4b
  SS58 Address:     5G9Wmx3mgRk7yKWPs421ZVuz1W34KxjKi3NDo7rVtdPMc6ju
```

### Soft Key Derivation

Likewise, you can derive a soft key child using a single `/` after the mnemonic phrase:

```bash
$ subkey inspect "spend report solution aspect tilt omit market cancel what type cave author/joe/polkadot/0"
Secret Key URI `spend report solution aspect tilt omit market cancel what type cave author/joe/polkadot/0` is account:
  Secret seed:      n/a
  Public key (hex): 0x06382e496cb8b1664501bbfcc8205c7b1ddc2402e32ef0479b18abdf326ca342
  Account ID:       0x06382e496cb8b1664501bbfcc8205c7b1ddc2402e32ef0479b18abdf326ca342
  SS58 Address:     5CCrqJffST8uh8RLn2sFJwg1JWs4HkKL88xmdZRdnHt8GDyA
```

Recall the address from the same seed phrase, `0x143fa4ecea108937a2324d36ee4cbce3c6f3a08b0499b276cd7adb7a7631a559`. We can use that to derive the same child address.

```bash
$ subkey inspect "5CXFinBHRrArHzmC6iYVHSSgY1wMQEdL2AiL6RmSEsFvWezd/joe/polkadot/0"
Public Key URI `5CXFinBHRrArHzmC6iYVHSSgY1wMQEdL2AiL6RmSEsFvWezd/joe/polkadot/0` is account:
  Network ID/version: substrate
  Public key (hex):   0x06382e496cb8b1664501bbfcc8205c7b1ddc2402e32ef0479b18abdf326ca342
  Account ID:         0x06382e496cb8b1664501bbfcc8205c7b1ddc2402e32ef0479b18abdf326ca342
  SS58 Address:       5CCrqJffST8uh8RLn2sFJwg1JWs4HkKL88xmdZRdnHt8GDyA
```

Note that the two addresses here match. This is not the case in hard key derivation.

### Putting it All Together

You can mix and match hard and soft key paths (although it doesn't make much sense to have hard paths as children of soft paths). For example:

```bash
$ subkey inspect "favorite liar zebra assume hurt cage any damp inherit rescue delay panic//joe//polkadot/0"
Secret Key URI `favorite liar zebra assume hurt cage any damp inherit rescue delay panic//joe//polkadot/0` is account:
  Public key (hex): 0xd09ab65c743e91b30d024469e8a8b823a3aa7e8f5b4791187adf531ac2af140f
  Account ID:       0xd09ab65c743e91b30d024469e8a8b823a3aa7e8f5b4791187adf531ac2af140f
  Address (SS58): 5GnDmyt7qnEN4esrLNRtjM7xHyRe4jQsbbAHpaNSsPNws7EB
```

The first two levels (`//joe//polkadot`) are hard-derived, while the leaf (`/0`) is soft-derived.

To use key derivation with a password protected key, add your password to the end:

```bash
$ subkey inspect "razor blouse enroll maximum lobster bacon raccoon ocean law question worry length/joe/polkadot/0///correct horse battery staple"
Secret Key URI `razor blouse enroll maximum lobster bacon raccoon ocean law question worry length/joe/polkadot/0///correct horse battery staple` is account:
  Secret seed:      n/a
  Public key (hex): 0x0816fe0689322e26cd2aa9c0dccb6c44851345e96f969ae85c8f1aec9fb4703d
  Account ID:       0x0816fe0689322e26cd2aa9c0dccb6c44851345e96f969ae85c8f1aec9fb4703d
  SS58 Address:     5CFK52zU59zUhC3s6mRobEJ3zm7JeXQZaS6ybvcuCDDhWwGG
```

Notice that the "address plus derivation path" produces the same address as the "mnemonic phrase plus derivation path plus password." As such, you can reveal your parent address and derivation paths without revealing your mnemonic phrase or password, while retaining control of all derived addresses.

## Well-known Keys

If you've worked with Substrate previously, you have likely encountered the ubiquitous accounts for Alice, Bob, and their friends. These keys are not at all private, but are useful for playing with Substrate without always generating new key pairs. You can inspect these "well-known" keys with `subkey`.

```bash
$ subkey inspect //Alice
Secret Key URI `//Alice` is account:
  Secret seed:      0xe5be9a5092b81bca64be81d212e7f2f9eba183bb7a90954f7b76361f6edb5c0a
  Public key (hex): 0xd43593c715fdd31c61141abd04a99fd6822c8558854ccde39a5684e7a56da27d
  Account ID:       0xd43593c715fdd31c61141abd04a99fd6822c8558854ccde39a5684e7a56da27d
  SS58 Address:     5GrwvaEF5zXb26Fz9rcQpDWS57CtERHpNehXCPcNoHGKutQY
```

There is nothing special about the name Alice. You can do the same trick with your own name. But remember that keys like this are _not_ secure, and are only useful for experimenting.

```bash
$ subkey inspect //Jimmy
Secret Key URI `//Jimmy` is account:
  Secret seed:      0x21f0a255194ed226b08966da162321570916d85073b6bdcae07d55001ab74b64
  Public key (hex): 0xd4acd1e68a71aeb85b885b8e20de1a20a62270eab49ca6cd224c194d0f135b75
  Account ID:       0xd4acd1e68a71aeb85b885b8e20de1a20a62270eab49ca6cd224c194d0f135b75
  SS58 Address:     5GsZM5RSjEgsStFPEtAVvtX2nTQk2HqtYR8BW3fyPBcCHabM
```

## Signing and Verifying Messages

### Signing

You can sign a message by passing the message to Subkey on STDIN. You can sign with either your seed or mnemonic phrase.

```bash
$ echo "test message" | subkey sign 0x554b6fc625fbea8f56eb56262d92ccb083fd6eaaf5ee9a966eaab4db2062f4d0
1e298698ed97654189ed082b3a19634c9cc2743e6e2e4089cc1759c959a8226d7709916041e195dd861a556ca1fde3d4305c00be3f08f72d369b83b9e3fa9b87
```

```bash
$ echo "test message" | subkey sign "spend report solution aspect tilt omit market cancel what type cave author"
069e02563e3d8d1f9af0d368a5420964643bf173da3794592c8602cf7feda149de041fa87c9137f2ccc7ec7e8bb9a13c4d3f0ba0729ba92eee50b4ecf772328e
```

### Verifying

Although the signatures above are different, they both verify with the address:

```bash
$ echo "msg" | subkey verify <signature> <SS58-address>

# Example
$ echo "test message" | subkey verify 1e298698ed97654189ed082b3a19634c9cc2743e6e2e4089cc1759c959a8226d7709916041e195dd861a556ca1fde3d4305c00be3f08f72d369b83b9e3fa9b87 5CXFinBHRrArHzmC6iYVHSSgY1wMQEdL2AiL6RmSEsFvWezd
Signature verifies correctly.
```

## Generating Node Keys

You can generate a node libp2p key by the following:

```bash
$ subkey generate-node-key <output-file>

# Example
$ subkey generate-node-key node-key
Qmb8aDXsAMoCnozJmUSaYzDTTxathFrsSU12A4owZ5K6V3
```

The peer ID is displayed and the actual key is saved in the `<output-file>`.

## More Subkey to Explore

* Learn more by running `subkey help` or see the [README](https://github.com/paritytech/substrate/tree/master/bin/utils/subkey).

* Key pairs can also be generated in the [PolkadotJS Apps UI](https://polkadot.js.org/apps/). Try creating keys with the UI and restoring them with `subkey` or vice versa.

* More about [different cryptographies and choosing between them](https://wiki.polkadot.network/docs/en/learn-cryptography).
