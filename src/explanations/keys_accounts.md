# Keys And Accounts

This page will describe some basic information on keys and accounts. For a more general and detailed explanation see [learn accounts](https://wiki.polkadot.network/docs/learn-accounts) on the polkadot wiki page.

## Encryption Schemes

| Name    | Type                                                                         |
| ------- | ---------------------------------------------------------------------------- |
| ed25519 | The vanilla ed25519 implementation using Schnorr signatures.                 |
| sr25519 | The Schnorrkel/Ristretto sr25519 variant using Schnorr signatures. (default) |
| ecdsa   | ECDSA signatures on secp256k1                                                |

## Session Key Types

| Name                | Type    |
|---------------------| ------- |
| grandpa             | ed25519 |
| authority_discovery | sr25519 |
| aura                | sr25519 |
| babe                | sr25519 |
| im_online           | sr25519 |
| para_validator      | sr25519 |
| para_assignment     | sr25519 |
| beefy               | ecdsa   |

## Key Generation and Inspection

You can use `polkadot-parachain keys` or the `subkey` command to generate and inspect keys.

Two important subcommands are:

- `generate` Create a new random account and print the private key data or save to a file.
- `inspect` View the account data for an account by passing a secret phrase or seed.

Some important options are:

- `--network` specify the network the keys will be used on, default is substrate.
- `--scheme` the scheme for the keys, default is sr25519.

## Generate

### Create a Random Account Key

```bash
$ polkadot-parachain key generate
Secret phrase:       test test test test test test test test test test test junk
  Network ID:        substrate
  Secret seed:       0x4ca479f5e0dc0ee04ebcbadb64c220267dad42b8cfa4da1f0874787523b4709c
  Public key (hex):  0xd000ac5048ae858aca2e6aa43e00661562a47026fe88ff83992430204a159752
  Account ID:        0xd000ac5048ae858aca2e6aa43e00661562a47026fe88ff83992430204a159752
  Public key (SS58): 5GmS1wtCfR4tK5SSgnZbVT4kYw5W8NmxmijcsxCQE6oLW6A8
  SS58 Address:      5GmS1wtCfR4tK5SSgnZbVT4kYw5W8NmxmijcsxCQE6oLW6A8
```

You can save it to a file using 

## Inspection

### Inspect Created Key

```bash
$ polkadot-parachain key inspect "test test test test test test test test test test test junk"
Secret phrase:       test test test test test test test test test test test junk
  Network ID:        substrate
  Secret seed:       0x4ca479f5e0dc0ee04ebcbadb64c220267dad42b8cfa4da1f0874787523b4709c
  Public key (hex):  0xd000ac5048ae858aca2e6aa43e00661562a47026fe88ff83992430204a159752
  Account ID:        0xd000ac5048ae858aca2e6aa43e00661562a47026fe88ff83992430204a159752
  Public key (SS58): 5GmS1wtCfR4tK5SSgnZbVT4kYw5W8NmxmijcsxCQE6oLW6A8
  SS58 Address:      5GmS1wtCfR4tK5SSgnZbVT4kYw5W8NmxmijcsxCQE6oLW6A8
```

### Inspect Created Key With Hard Derivation //Stash//0

```bash
$ polkadot-parachain key inspect " test test test test test test test test test test test junk//Stash//0"
Secret Key URI ` test test test test test test test test test test test junk//Stash//0` is account:
  Network ID:        substrate
  Secret seed:       0x88f170a1438f83de6c2b845282ebb577dfcaee4c896655e64777ba220b3ad33d
  Public key (hex):  0xa2a73aaf576ed83371c92642a80706bf4b007c40f689d649aa7b075833d7242c
  Account ID:        0xa2a73aaf576ed83371c92642a80706bf4b007c40f689d649aa7b075833d7242c
  Public key (SS58): 5FjyHtMBm7kArqgSX52DAQvZeJ2Tq5sQBwFfSmZ3NWMHs4Be
  SS58 Address:      5FjyHtMBm7kArqgSX52DAQvZeJ2Tq5sQBwFfSmZ3NWMHs4Be
```

## Managing a node session keys

In Polkadot, **Session Keys** refer to the hot keys set on a node to perform various network operations, such as collating or validating blocks.

### Create new session keys on a node via the `author_rotateKeys` RPC method

On the remote machine hosting the node, you can create new session keys with the following commmand:

```bash
curl -H "Content-Type: application/json" -d '{"id":1, "jsonrpc":"2.0", "method": "author_rotateKeys", "params":[]}' http://localhost:9944
```

The response output will then show the hex-encoded concatenation of the session public keys. This is intended to be submitted on-chain using the [session.setKeys extrinsics](https://wiki.polkadot.network/docs/maintain-guides-how-to-validate-polkadot#submitting-the-setkeys-transaction).

### Insert a specific key onto a node keystore via the `author_insertKey` RPC method

To inject a key via RPC, use the following command:

```bash
curl -H "Content-Type: application/json" \ --data '{ "jsonrpc":"2.0", "method":"author_insertKey", "params":["'"${KEY_TYPE}"'", "'"${KEY_SEED}"'"],"id":1 }' http://localhost:9933
```

Set:
* `KEY_TYPE` to a session key type (eg. `babe`, `grandpa`)
* `KEY_SEED` to a secret seed or private key

### Insert a specific key onto a node keystore using the node binary

To inject a key via the node binary, use the following command:

```bash
polkadot-parachain key insert -d <my_chain_folder> --key-type ${KEY_TYPE} --scheme ${KEY_SCHEME} --suri ${KEY_FILE}
```

* `KEY_TYPE`: a session key type (eg. `babe`, `grandpa`)
* `KEY_SCHEME`: an encryption scheme (eg. `sr25519`, `ed25519`, `ecdsa`)
* `KEY_FILE`: a plain text file which contains the seed or private key
