---
CIP: TODO
Title: Catalyst Registration Transaction Metadata Format
Authors: Sebastien Guillemot <sebastien@emurgo.io>, Rinor Hoxha <rinor.hoxha@iohk.io>, Mikhail Zabaluev <mikhail.zabaluev@iohk.io>
Comments-URI: https://forum.cardano.org/t/cip-catalyst-registration-metadata-format/44038
Status: Draft
Type: Standards
Created: 2020-01-05
License: CC-BY-4.0
---

## Abstract

Cardano uses a sidechain for its treasury system. One needs to "register" to participate on this sidechain by submitting a registration transaction on the mainnet chain. This CIP details the registration transaction format.

## Motivation

Cardano uses a sidechain for its treasury system ("Catalyst"). One of the desirable properties of this sidechain is that even if its safety is compromised, it doesn't cause loss of funds on the main Cardano chain. To achieve this, instead of using your wallet's recovery phrase on the sidechain, we need to use a brand new "voting key".

However, since 1 ADA = 1 vote, a user needs to associate their mainnet ADA to their new voting key. This can be achieved through a registration transaction.

We therefore need a registration transaction that serves three purposes:

1. Registers a "voting key" to be included in the sidechain
2. Associates mainnet ADA to this voting key
3. Declare an address to receive Catalyst rewards

## Specification

### Voting key format

A voting key is simply an ED25519 key. How this key is created is up to the wallet.

### Interim registration method: associating stake with a voting key

This method has been used for Fund 2 and will be used for Fund 3.
For future fund iterations, a new method making use of time-lock scripts will
be used as described in the section below.

Recall: Cardano uses the UTXO model so to completely associate a wallet's balance with a voting key (i.e. including enterprise addresses), we would need to associate every payment key to a voting key individually. Although there are attempts at this (see [CIP8](../CIP-0008/CIP-0008.md)), the resulting data structure is a little excessive for on-chain metadata (which we want to keep small)

Given the above, we choose to only associate staking keys with voting keys. Since most Cardano wallets only use base addresses for Shelley wallet types, in most cases this should perfectly match the user's wallet.

#### Registration metadata format with a stake public key

A Catalyst registration transaction for funds 2 and 3 is a regular Cardano transaction with a specific transaction metadata associated with it.

Notably, there should be three entries inside the metadata map:

Voting key registration
```
61284: {
  // voting_key - CBOR byte array
  1: "0xa6a3c0447aeb9cc54cf6422ba32b294e5e1c3ef6d782f2acff4a70694c4d1663",
  // stake_pub - CBOR byte array
  2: "0xad4b948699193634a39dd56f779a2951a24779ad52aa7916f6912b8ec4702cee",
  // address - CBOR byte array
  3: "0x00588e8e1d18cba576a4d35758069fe94e53f638b6faf7c07b8abd2bc5c5cdee47b60edc7772855324c85033c638364214cbfc6627889f81c4"
}
```

Signing the voting public key with the staking private key
```
61285: {
  // signature - ED25119 signature CBOR byte array
  1: "0x8b508822ac89bacb1f9c3a3ef0dc62fd72a0bd3849e2381b17272b68a8f52ea8240dcc855f2264db29a8512bfcd522ab69b982cb011e5f43d0154e72f505f007"
}
```

### Evolved registration method: locking funds for the governance period

Starting from Fund 4, it is planned to use a time-lock script (documentation TBA) to commit ADA on the mainnet for the duration of a voting period. The voter registration metadata in this method does not need an association
with the staking key.

#### Registration metadata format with token locking

A Catalyst registration transaction is a Cardano transaction with an output locked by a multisig script, with the time locks preventing spending of the output during the voting period, and specific transaction metadata providing the voting public key and the address for receiving rewards.

There should be two entries inside the metadata map:

Voting key registration
```
61284: {
  // voting_key - CBOR byte array
  1: "0xa6a3c0447aeb9cc54cf6422ba32b294e5e1c3ef6d782f2acff4a70694c4d1663",
  // address - CBOR byte array
  3: "0x00588e8e1d18cba576a4d35758069fe94e53f638b6faf7c07b8abd2bc5c5cdee47b60edc7772855324c85033c638364214cbfc6627889f81c4"
}
```

### Metadata schema

The two formats above can be described by the following CDDL definition, where optional fields are deprecated in the voter registration metadata format used since Fund 4.

```
registration_cbor = {
  61284: key_registration,
  ? 61285: registration_signature
}

$voting_pub_key /= bytes .size 32
$staking_pub_key /= bytes .size 32
$ed25519_signature /= bytes .size 64
$address /= bytes

key_registration = {
  1 : $voting_pub_key,
  ? 2 : $staking_pub_key,
  3 : $address
}

registration_signature = {
  1 : $ed25519_signature
}
```

Here an example using CBOR diagnostic notation

```
{
  61284: {
    1: h'8253C95609BC62C0443276FE2A1872B87CB11C06185FFDBB56C7CE8352EEF2A3',
    3: h'00588E8E1D18CBA576A4D35758069FE94E53F638B6FAF7C07B8ABD2BC5C5CDEE47B60EDC7772855324C85033C638364214CBFC6627889F81C4'
  },
}
```

## Changelog

Fund 3 added the `address` inside the `key_registration` field.

Since Fund 4, `registration_signature` and the `staking_pub_key` entry inside
the `key_registration` field are deprecated and should no longer be used.

## Copyright

This CIP is licensed under [CC-BY-4.0](https://creativecommons.org/licenses/by/4.0/legalcode)
