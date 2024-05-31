# Guide: Deploying a Parachain Network

This guide demonstrates the deployment of a parachain test network composed of 2 collators (nodes authoring blocks) and 1 RPC node.

## Preparations

### Hardware
To run your nodes, you'll need suitable machines. For this example network, you will need 3 machines.
For a testnet, medium-sized virtual machines (2 to 4 cores) will suffice. However, for mainnet nodes, it is recommended to follow the ["validator reference hardware" as detailed in the Polkadot Wiki](https://wiki.polkadot.network/docs/maintain-guides-how-to-validate-polkadot#reference-hardware).

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

Then publish the node binary present in `target/releases` somewhere and note down the public URL. One option to One option to do this is to add it as a [Github release asset](https://docs.github.com/en/repositories/releasing-projects-on-github/managing-releases-in-a-repository).
In this guide, we will use the polkadot-parachain binary released on the paritytech/polkadot-sdk. This binary is used to deploy system parachain nodes, such as asset-hub and bridge-hub and available at `https://github.com/paritytech/polkadot-sdk/releases/latest/download/polkadot-parachain`.

### Prepare the docker image for use with Kubernetes

To deploy your network with the [node helm-chart](https://github.com/paritytech/helm-charts/tree/main/charts/node), you will need to have a node docker image published in a registry
In this guide, we will use the [official `paritech/polkadot-parachain` published on Docker Hub](https://hub.docker.com/r/parity/polkadot-parachain/tags).

## Generate parachain private keys 

### Generate static node keys (aka network keys)

Node keys are used to identify nodes on the P2P network with a unique PeerID.
To ensure this identifier persists across restarts, it is highly recommended to generate a static key for all nodes.
This practice is particularly important for bootnodes, as other nodes in the network use these bootnodes to bootstrap their connections to the network.

To generate a static node key:

```
$ polkadot-parachain key generate-node-key
# example output
12D3KooWExcVYu7Mvjd4kxPVLwN2ZPnZ5NyLZ5ft477wqzfP2q6E # PeerId (hash of public node key)
d5e120e30dfb0eac39b1a6727d81c548e9c6b39ca97e565438a33d87726781a6% # private node key, do not copy the percent sign
```

You can use this bash script to generate several keys in one go. Save this file to `generate-node-keys.sh`
```
#!/bin/bash
# $1: chain name / name of the folder to create in keys will be saved
# $2: number of keys to generate

mkdir -p $1

let "max_key_id=$2-1"
echo "generating $2 node-keys for chain $1"

for i in ` seq -w 0 $max_key_id `; do
  # generate node key
  subkey generate-node-key --file $1/node-key-$i
  subkey inspect-node-key --file $1/node-key-$i > $1/node-key-$i-id
  # save node key info
  echo "$1-node-$i:"
  echo "  node_parachain_p2p_public_key: $(cat $1/node-key-$i-id)"
  echo "  node_parachain_p2p_private_key: $(cat $1/node-key-$i)"
done
```

```
chmod +x generate-node-keys.sh
./generate-node-keys.sh my-parachain 3
```

### Generate keys for your collators (account+aura keys)

For parachains using the collatorSelection pallet to manage their collator set, you will need to generate a set of keys for each collator:

* Collator Account: a regular substrate account
* Aura keys (part of the collator “session keys”)

In this example, we will use the same seed for both. Use the following command generate a secret seed for each collator:

```
polkadot-parachain key generate
```

You can derive public keys and account IDs from an existing seed by using:
```
$ ./polkadot-parachain key inspect "//Alice"

Secret Key URI `//Alice` is account:
  Network ID:        substrate
  Secret seed:       0xe5be9a5092b81bca64be81d212e7f2f9eba183bb7a90954f7b76361f6edb5c0a
  Public key (hex):  0xd43593c715fdd31c61141abd04a99fd6822c8558854ccde39a5684e7a56da27d
  Account ID:        0xd43593c715fdd31c61141abd04a99fd6822c8558854ccde39a5684e7a56da27d
  Public key (SS58): 5GrwvaEF5zXb26Fz9rcQpDWS57CtERHpNehXCPcNoHGKutQY
  SS58 Address:      5GrwvaEF5zXb26Fz9rcQpDWS57CtERHpNehXCPcNoHGKutQY
```

## Create your Network Chainspec

When launching a new parachain network, customizing the chainspec (chain specification) is a crucial step.
Here are the general steps to customize your chainspec:

### Generate a plain Chainspec

```
./polkadot-parachain build-spec --disable-default-bootnode > chainspec.plain.json
```

If you want to select a specific built-in chainspec of your binary, add `--chain chain-name` to your command.

### Customize the Plain Chainspec

To customize the chainspec for your parachain network, you'll need to edit the following fields in the `chainspec.plain.json` file:

* Set `name`, `id`, and `protocolId` to unique values:

```json
{
"name": "UniqueParachainName",
"id": "unique-parachain-id",
"protocolId": "unique-protocol-id",
...
}
```

* Set `relay_chain` to rococo:

```json
{
"relay_chain": "rococo",
...
}
```

* Set `chainType` to Live:

```json
{
"chainType": "Live",
...
}
```

* Set your ParaID in both `para_id` and `genesis.runtimeGenesis.patch.parachainInfo.parachainId`:

```json
{
  "para_id": <your-parachain-id>,
  "genesis": {
    "runtime": {
      "runtimeGenesis": {
        "patch": {
          "parachainInfo": {
            "parachainId": <your-parachain-id>
        }
      }
    }
  }
},
...
}
```

* Set `sudo.key` to your desired sudo account:

```json
{
  "genesis": {
    "runtime": {
      "sudo": {
        "key": "<your-sudo-account>"
      }
    }
  },
  ...
}
```

You will also need specific customization relevant to your nodes:

* Set `bootNodes` addresses, any node which has a public IP or DNS can be a bootnode:

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

* Set your genesis parachain keys

```json
{
    "collatorSelection": {
     "candidacyBond": 16000000000, # For non-invulnerables collators
     "invulnerables": [
       "5GrwvaEF5zXb26Fz9rcQpDWS57CtERHpNehXCPcNoHGKutQY", # Collator 1 Address
       "5FHneW46xGXgs5mUiveU4sbTyGBzmstUspZC92UhjJM694ty" # Collator 2 Address
     ]
    },
    ...
    "session": {
     "keys": [
       [
         "5GrwvaEF5zXb26Fz9rcQpDWS57CtERHpNehXCPcNoHGKutQY", # Collator Address
         "5GrwvaEF5zXb26Fz9rcQpDWS57CtERHpNehXCPcNoHGKutQY", # Collator Controller Address (for legacy controller/stash separation, just set this to the same as above)
         {
           "aura": "5GrwvaEF5zXb26Fz9rcQpDWS57CtERHpNehXCPcNoHGKutQY" # Collator aura public key
         }
       ],
       [
         ...
       ]
     ]
},
```

### Convert your Plain Chainspec to Raw

To initialize the genesis storage for your chain, you need convert your chainspec from plain to raw format.
This process transforms the human-readable keys in the plain chainspec into actual storage keys and defines a unique genesis block.

A unique [raw chainspec](https://docs.substrate.io/build/chain-spec/#raw-chain-specifications) can be created from the plain chainspec with this command:

```
polkadot-parachain build-spec --chain chainspec.plain.json --raw >  chainspec.raw.json
```

⚠️ Only use the raw chainspec to launch your chain, not the plain chainspec.


### (Optional) Dry-run your Parachain Network Locally

You should now have everything ready to launch your network locally to validate that everything is properly set up.

* Start your node (we use the `--tmp` flag to prevent the node database files from being persisted to disk):

```
./polkadot-parachain --chain chainspec.raw.json --tmp
````
* Connect to it with [Polkadot.js Apps](https://polkadot.js.org/apps/?rpc=ws%3A%2F%2F127.0.0.1%3A9944) on `ws://127.0.0.1:9944`.
* You can inspect the [chain state in Polkadot.js Apps](https://polkadot.js.org/apps/?rpc=ws%3A%2F%2F127.0.0.1%3A9944#/chainstate) to verify that everything is in order for the launch.

Note: if you look at the node logs, it should be starting to sync the relay-chain (Rococo in our case). You don’t have to wait until it is fully synced.

## Deploy your nodes

You can use any method you choose to set up your nodes on your machines, we recommend either Ansible 

### Deploy your nodes with Ansible

Clone the project at [paritytech/parachain-deployment-quickstart](https://github.com/paritytech/parachain-deployment-quickstart/) and follow instructions in the `ansible` folder.

### Deploy your nodes with Kubernetes

Clone the project at [paritytech/parachain-deployment-quickstart](https://github.com/paritytech/parachain-deployment-quickstart/) and follow instructions in the `kubernetes` folder.

## Register and activate your Parachain on the Relaychain (Rococo)

### Reserve a ParaId on Rococo

Note: although it is possible to use specific UIs for registering your parachain, this guide only documents how to do it by submitting extrinsics directly through the Polkadot.js Console.

To get reserve a ParaId for your parachain on Rococo, navigate to the [Polkadot.js Apps interface](https://polkadot.js.org/apps/?rpc=wss%3A%2F%2Frococo-rpc.polkadot.io).

* Ensure you are connected to the Rococo network by selecting the appropriate RPC endpoint (`wss://rococo-rpc.polkadot.io`).
* Go to the "Developer" tab and select "Extrinsics".
* Choose `registrar.reserve` from the dropdown menu and execute it with your account.
* Check the included extrinsic result in the block to find your reserved `para_id` and note it down.

### Register parachain genesis code and state on relay-chain

You can export the genesis runtime (WASM code) and state files from your chainspec. Then, these files can be registered on the relaychain to initialize your parachain.

* Export the genesis state:
```
./polkadot-parachain export-genesis-state --chain chainspec.raw.json > genesis_state_head
```
* Export the genesis runtime:
```
./polkadot-parachain export-genesis-wasm --chain chainspec.raw.json > genesis_wasm_code
```
* Register your parachain genesis configuration on the relay-chain by executing the registrar.register  extrinsic on Rococo:
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
- On-demand coretime: lets users buy coretime by the block. Useful for parachains which don’t require continuous block production
- Bulk coretime: an allocation of uninterrupted 28 days (default region length) of coretime (possibly split in timeslices)

For more information, please refer to the [Parachain Coretime Guide](https://wiki.polkadot.network/docs/learn-guides-coretime-parachains).

### Reserve Bulk Coretime

* Go on [Lastic Rococo Coretime Sales page](https://www.lastic.xyz/rococo/bulkcore1) and buy 1 core
* Go on [Lastic Rococo Homepage](https://www.lastic.xyz/rococo/my-cores), click on the “core nb X” that you own to show its dedicated page, then click "assign core".
  Assign it to your ParaID and Finality: `Final`

Note: any account with enough funds can buy and assign coretime for a parachain.


### Renew Bulk Coretime

TODO