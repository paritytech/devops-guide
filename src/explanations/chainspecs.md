# Custom Chain - Chainspecs

This section will explain what is required to build a custom chain spec for your own network.
For more in depth explanations on how chainspecs works internally, refer to the [Polkadot-sdk Chainspec docs](https://paritytech.github.io/polkadot-sdk/master/polkadot_sdk_docs/reference_docs/chain_spec_genesis/index.html)

## Create Chain Spec

Create a non-raw chainspec for your network with:

```bash
./binary build-spec –chain chain-name --disable-default-bootnode > custom-chainspec.json
```

**Important Fields**:

| Field                  | Purpose                                                              |
|------------------------|----------------------------------------------------------------------|
| Name                   | Name of the network                                                  |
| id                     | Id of network, also used for the filesystem path                     |
| bootNodes              | List of multiaddr’s of bootnodes for the network                     |
| protocolId             | A unique identifier for the protocol                                 |
| para_id                | (Parachains only) numerical ID of the parachain slot                 |
| relay_chain            | (Parachains only) Id of the relaychain to connect to                 |
| genesis.runtimeGenesis | (Plain chainspec only) Configuration of the genesis runtime          |
| genesis.raw            | (Raw chainspec only) Serialization of the raw of the genesis storage |

Once the values have been updated this chainspec should be converted in raw format using the command:

```bash
polkadot  build-spec –chain custom-chainspec.json –raw > custom-chainspec-raw.json
```