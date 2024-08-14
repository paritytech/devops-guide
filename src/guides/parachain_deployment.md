# Guide: Deploying a Parachain Network

This guide demonstrates the deployment of a parachain test network composed of 2 collators (nodes authoring blocks) and 1 RPC node.
We are using Rococo as an example, but this approach would work similarly for any Relaychain, whether it is a testnet (Westend, Rococo, Paseo) or a mainnet (Polkadot, Kusama).

## Preparations

### Hardware

For this example network, you will need 3 machines.
The specifications of these machines will depend on your intended usage. For a testnet, medium-sized virtual machines with 2 to 4 cores will suffice. However, for mainnet nodes, it is recommended to follow the ["validator reference hardware" as detailed in the Polkadot Wiki](https://wiki.polkadot.network/docs/maintain-guides-how-to-validate-polkadot#reference-hardware).

Requirements:

* The machines should have a public IP and allow network access on their P2P ports (defaults 30333 and 30334) as well as the RPC port for the RPC node (9944 for ws or 443 for wss).
* The machine should have a big enough disk to host the relay-chain pruned database (>250 GB for Rococo)
* You should have obtained SSH access to these machines.

### Parachain binary or docker image

### Prepare the binary for use with Ansible

To deploy your network with the [Ansible node role](https://github.com/paritytech/ansible-galaxy/blob/main/roles/node), you will need to have the node binary available from a public URL.
If this binary is not already available for your parachain, you will need to build it yourself using the following command:

```
cargo build --release
```

Then publish the node binary present in `target/releases` somewhere and note down the public URL. One option to do this is to add it as a [GitHub release asset](https://docs.github.com/en/repositories/releasing-projects-on-github/managing-releases-in-a-repository).
In this guide, we will use the `parachain-template-node` that you can get from [paritytech/polkadot-sdk-parachain-template's releases assets](https://github.com/paritytech/polkadot-sdk-parachain-template/releases).

To deploy system parachain nodes, such as asset-hub and bridge-hub, you should use the `polkadot-parachain` binary published on [https://github.com/paritytech/polkadot-sdk/releases/latest/](https://github.com/paritytech/polkadot-sdk/releases/latest).

### Prepare the docker image for use with Kubernetes

To deploy your network with the [node helm-chart](https://github.com/paritytech/helm-charts/tree/main/charts/node), you will need to have a node docker image published to a public registry.

## Generate parachain private keys

In this section, we will use the [`subkey`](./subkey.md) utility.

### Generate static node keys (aka network keys)

Node keys are used to identify nodes on the P2P network with a unique PeerID.
To ensure this identifier persists across restarts, it is highly recommended to generate a static network key for all nodes.
This practice is particularly important for bootnodes, which have publicly listed addresses that are used by other nodes to bootstrap their connections to the network.

To generate a static node key:

```
subkey generate-node-key --file node.key
# example output
12D3KooWExcVYu7Mvjd4kxPVLwN2ZPnZ5NyLZ5ft477wqzfP2q6E # PeerId (hash of public node key)
cat node.key
d5e120e30dfb0eac39b1a6727d81c548e9c6b39ca97e565438a33d87726781a6% # private node key, do not copy the percent sign
```

### Generate keys for your collators (account+aura keys)

For parachains using the collatorSelection pallet to manage their collator set, you will need to generate a set of keys for each collator:

* Collator Account: a regular substrate account
* Aura keys (part of the collator “session keys”)

In this example, we will use the same seed for both. Use the following command generate a secret seed for each collator:

```
subkey generate
```

You can derive public keys and account IDs from an existing seed by using:
```
subkey inspect "<secret-seed>"
# example output
Secret Key URI `//Alice` is account:
  Network ID:        substrate
  Secret seed:       0xe5be9a5092b81bca64be81d212e7f2f9eba183bb7a90954f7b76361f6edb5c0a
  Public key (hex):  0xd43593c715fdd31c61141abd04a99fd6822c8558854ccde39a5684e7a56da27d
  Account ID:        0xd43593c715fdd31c61141abd04a99fd6822c8558854ccde39a5684e7a56da27d
  Public key (SS58): 5GrwvaEF5zXb26Fz9rcQpDWS57CtERHpNehXCPcNoHGKutQY
  SS58 Address:      5GrwvaEF5zXb26Fz9rcQpDWS57CtERHpNehXCPcNoHGKutQY
```

## Reserve a ParaId on Rococo

Note: although it is possible to use specific UIs for registering your parachain, this guide only documents how to do it by submitting extrinsics directly through the Polkadot.js Console.

To get reserve a ParaId for your parachain on Rococo, navigate to the [Polkadot.js Apps interface](https://polkadot.js.org/apps/?rpc=wss%3A%2F%2Frococo-rpc.polkadot.io).

* Ensure you are connected to the Rococo network by selecting the appropriate RPC endpoint (`wss://rococo-rpc.polkadot.io`).
* Go to the "Developer" tab and select "Extrinsics".
* Choose `registrar.reserve` from the dropdown menu and execute it with your account.
* Check the included extrinsic result in the block to find your reserved `para_id` and note it down.


## Create your Network Chainspec

When launching a new parachain network, customizing the chainspec (chain specification) is a crucial step.

### Export your runtime file

First export your runtime file from your node binary (here `--raw` is used to export the output as binary not hex string):

```
parachain-template-node export-genesis-wasm --raw > runtime.wasm
```

If you want to select a specific built-in runtimle of your binary, add `--chain chain-name` to your command.

### Prepare your genesis patch config

Save the following to `genesis.patch.json` (replace keys and configuration with your own):
```json
{
  "balances": {
    "balances": [
      [
        "5GrwvaEF5zXb26Fz9rcQpDWS57CtERHpNehXCPcNoHGKutQY",
        1152921504606846976
      ],
      [
        "5FHneW46xGXgs5mUiveU4sbTyGBzmstUspZC92UhjJM694ty",
        1152921504606846976
      ]
    ]
  },
  "collatorSelection": {
    "candidacyBond": 16000000000,
    "invulnerables": [
      "5GrwvaEF5zXb26Fz9rcQpDWS57CtERHpNehXCPcNoHGKutQY",
      "5FHneW46xGXgs5mUiveU4sbTyGBzmstUspZC92UhjJM694ty"
    ]
  },
  "session": {
    "keys": [
      [
        "5GrwvaEF5zXb26Fz9rcQpDWS57CtERHpNehXCPcNoHGKutQY",
        "5GrwvaEF5zXb26Fz9rcQpDWS57CtERHpNehXCPcNoHGKutQY",
        {
          "aura": "5GrwvaEF5zXb26Fz9rcQpDWS57CtERHpNehXCPcNoHGKutQY"
        }
      ],
      [
        "5FHneW46xGXgs5mUiveU4sbTyGBzmstUspZC92UhjJM694ty",
        "5FHneW46xGXgs5mUiveU4sbTyGBzmstUspZC92UhjJM694ty",
        {
          "aura": "5FHneW46xGXgs5mUiveU4sbTyGBzmstUspZC92UhjJM694ty"
        }
      ]
    ]
  },
  "parachainInfo": {
    "parachainId": 4435
  },
  "polkadotXcm": {
    "safeXcmVersion": 4
  },
  "sudo": {
    "key": "5GrwvaEF5zXb26Fz9rcQpDWS57CtERHpNehXCPcNoHGKutQY"
  }
}
```

In this example:

* `balances`: initial account balances
* `collatorSelection`: configure the collatorSelection pallet properties, in this example we set Alice and Bob as initial invulnerable collators.
* `session.keys`: initial session keys
* `parachainInfo.parachainId`: parachain ID
* `sudo.keys`: initial sudo key account

Note that:
- `5GrwvaEF5zXb26Fz9rcQpDWS57CtERHpNehXCPcNoHGKutQY`: Alice's account address
- `5FHneW46xGXgs5mUiveU4sbTyGBzmstUspZC92UhjJM694ty`: Bob's account address

### Generate a customized plain Chainspec

With the **chain-spec-builder** utility, we can generate a chainspec file using only the runtime wasm file 
on which we will apply a patch describing the customized genesis config we want to apply

Download the chain-spec-builder binary from the latest [polkadot-sdk releases](https://github.com/paritytech/polkadot-sdk/releases/latest).
Then execute it, taking as input the runtime and genesis patch files:

```bash
chain-spec-builder -c chainspec.plain.json create -n "Test Parachain" -i test-parachain -t live -r runtime.wasm patch genesis.patch.json
```

To work properly as a parachain chainspec, add the following fields to your `chainspec.plain.json`:
```
{
  "protocolId": "template-local",
  "properties": {
    "ss58Format": 42,
    "tokenDecimals": 12,
    "tokenSymbol": "UNIT"
  },
  "para_id": 4435,
  "relay_chain": "rococo",
  ...
```

You also need to set your `bootNodes` addresses, any node which has a public IP or DNS can be a bootnode:

```json
{
  "bootNodes": [
    # eg. IP bootnode
    "/ip4/<Node-Public-IP>/tcp/30333/p2p/<Node-ID>"
    # eg. DNS bootnode
    "/dns/<Node-Public-DNS>/tcp/30333/p2p/<Node-ID>",
    # (Optional) WSS Bootnodes (for light clients, requires a TLS cert, see https://wiki.polkadot.network/docs/maintain-bootnode)   
    "/dns/<Node-Public-DNS>/tcp/443/wss/p2p/<Node-ID>",
  ...
}
```

### Convert your Plain Chainspec to Raw

To initialize the genesis storage for your chain, you need convert your chainspec from plain to raw format.
This process transforms the human-readable keys in the plain chainspec into actual storage keys and defines a unique genesis block.

A unique [raw chainspec](https://docs.substrate.io/build/chain-spec/#raw-chain-specifications) can be created from the plain chainspec with this command:

```
parachain-template-node build-spec --chain chainspec.plain.json --raw > chainspec.raw.json
```

⚠️ Only use the raw chainspec to launch your chain, not the plain chainspec.


### (Optional) Dry-run your Parachain Network Locally

You should now have everything ready to launch your network locally to validate that everything is properly set up.

* Start your node (we use the `--tmp` flag to prevent the node database files from being persisted to disk):

```
parachain-template-node --chain chainspec.raw.json --tmp
````
* Connect to it with [Polkadot.js Apps](https://polkadot.js.org/apps/?rpc=ws%3A%2F%2F127.0.0.1%3A9944) on `ws://127.0.0.1:9944`.
* You can inspect the [chain state in Polkadot.js Apps](https://polkadot.js.org/apps/?rpc=ws%3A%2F%2F127.0.0.1%3A9944#/chainstate) to verify that everything is in order for the launch.

Note: if you look at the node logs, it should be starting to sync the relay-chain (Rococo in our case). You don’t have to wait until it is fully synced.

## Deploy your nodes

You can use any method you choose to set up your nodes on your machines, we recommend either Ansible or Kubernetes.

### Deploy your nodes with Ansible

Clone the project at [paritytech/parachain-deployment-quickstart](https://github.com/paritytech/parachain-deployment-quickstart/) and follow instructions in the `ansible` folder.

### Deploy your nodes with Kubernetes

Clone the project at [paritytech/parachain-deployment-quickstart](https://github.com/paritytech/parachain-deployment-quickstart/) and follow instructions in the `kubernetes` folder.

## Register and activate your Parachain on the Relaychain

### Register parachain genesis code and state on relay-chain

You can export the genesis runtime (WASM code) and state files from your chainspec. Then, these files can be registered on the relaychain to initialize your parachain.

* Export the genesis state:
```
parachain-template-node export-genesis-state --chain chainspec.raw.json > genesis_state_head
```
* Export the genesis runtime:
```
parachain-template-node export-genesis-wasm --chain chainspec.raw.json > genesis_wasm_code
```
* Register your parachain genesis configuration on the relay-chain by executing the `registrar.register` extrinsic on Rococo:
  - `id`: your parachain ID
  - `genesisHead`: select the `genesis_state_head` file
  - `validationCode`: select the `genesis_wasm_code` file

### Obtain on-demand coretime to produce your first block

* Execute extrinsic: `onDemandAssignmentProvider.placeOrderAllowDeath` or `onDemandAssignmentProvider.placeOrderKeepAlive`:
  - `maxAmount`: `10000000000000` (13 zeros, ie. 10 ROC)
  - `paraId`: your parachain ID

After executing this, you should have successfully produced your first block !

```
INFO tokio-runtime-worker substrate: [Parachain] ✨ Imported #1 (0xa075…10d6)
```

## Manage Coretime for your chain

To allow your parachain to produce blocks it needs to be allocated [coretime](https://wiki.polkadot.network/docs/learn-agile-coretime) (time allocated for utilizing a core) on the relaychain.
What we mean by "core" is the ability of the relaychain (and its validators) to validate the new blocks of a parachain (received from the collators) so that they are “included” in Polkadot as finalized parachain blocks.

There are 2 types of coretime:
- **On-demand coretime**: lets users buy coretime by the block. Useful for parachains which don’t require continuous block production
- **Bulk coretime**: an allocation of uninterrupted 28 days (default region length) of coretime (possibly split in timeslices)

For more information, please refer to the [Parachain Coretime Guide](https://wiki.polkadot.network/docs/learn-guides-coretime-parachains).

### Reserve Bulk Coretime

* Go on [Lastic Rococo Coretime Sales page](https://www.lastic.xyz/rococo/bulkcore1) and buy 1 core
* Go on [Lastic Rococo Homepage](https://www.lastic.xyz/rococo/my-cores), click on the “core nb X” that you own to show its dedicated page, then click "assign core".
  Assign it to your ParaID and Finality: `Final`

Note: any account with enough funds can buy and assign coretime for a parachain.

### Renew Bulk Coretime

TODO
