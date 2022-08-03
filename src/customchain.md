Custom Chain
=============

This section will explain what is required to build a custom chain spec for your own dev network.

## Create Chain Spec

Create a non-raw chainspec for your network with:

```bash

./binary build-spec –chain chain-name --disable-default-bootnode > custom-chainspec.json

```

**Important Fields**:

| Field | Purpose |
| ------- | ----------- |
| Name | Name of the network |
| id | Id of network, also used for the filesystem path |
| bootNodes | List of multiaddr’s of bootnodes for the network |
| telemetryEndpoints | List of telemetry endpoints to contact by default |
| staking.stakers | List of initial validators |
| session.keys | List of initial session keys for validators |



Once the values have been updated this chainspec should be converted in raw format using the command:

```bash

polkadot  build-spec –chain custom-chainspec.json –raw > custom-chainspec-raw.json

```


## Inject initial validator keys

There are two main ways to inject the initial validator keys:

### CLI

Example to insert one ed25519 and one sr25519 key.

```bash
polkadot key insert -d <my_chain_folder> --key-type gran --scheme ed25519 --suri <my_key_suri>
polkadot key insert -d <my_chain_folder> --key-type babe --scheme sr25519 --suri <my_key_suri>

```

### RPC

Example to inject key via RPC, edit the KEY_TYPE sand KEY_SEED:

```bash

curl -H "Content-Type: application/json" \ --data '{ "jsonrpc":"2.0", "method":"author_insertKey", "params":["'"${KEY_TYPE}"'", "'"${KEY_SEED}"'"],"id":1 }' http://localhost:9933

```

