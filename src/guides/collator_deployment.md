# Introduction
Collators are members of the network that maintain the parachains they take part in. Collators produce the state transition proof for relay chain validators but also author the blocks for the parachain and pass them to relay chain. There are some requirements that need to be considered prior to becoming a collator candidate including machine, bonding, account, and community requirements. For more info regarding collators, reference [collator section](https://wiki.polkadot.network/docs/learn-collator) on Polkadot wiki page.

This doc demonstrates how to add new collators to an already running parachain.

# Hardware Requirements
Make sure your instance has at least the minimum recommended harware requirements:
- CPU: 2 Cores - Intel Xeon E-2386/2388 or Ryzen 9 5950x/5900x
- Memory: 8GB 
- Disk: Depending on the chain size - 200GB SSD/nvme

# Launch the Collator 
1- Generate an Account
2- Run the Collator
3- Bind The Account With Collator's Session Keys
4- Bond Funds


## 1- Create an Account
This account is used to hold funds and submit the extrinsics to bond and register your collator as candidate.

```
> subkey generate
Secret phrase:       crane material doll wife text jaguar cross donor crane caught soft permit
  Network ID:        substrate
  Secret seed:       0x79613c5ccb1c8aa9c21e1ddb2fbf65c8e979649cd52cfe041391c9f9d07ec3e9
  Public key (hex):  0x4a503b20b3ada9ff3bf985271b5756e650a83ae47e44a7552dc2415ce8060a52
  Account ID:        0x4a503b20b3ada9ff3bf985271b5756e650a83ae47e44a7552dc2415ce8060a52
  Public key (SS58): 5Dk9EY7iUbjLX6fTDnbFZVGdnN7v2NgQtxj4Y7YjjZXgpCjE
  SS58 Address:      5Dk9EY7iUbjLX6fTDnbFZVGdnN7v2NgQtxj4Y7YjjZXgpCjE
```

Store the secret phrase in a secure place. This account will be used on PolkadotJS app to bind you collator session keys to it.

## 2- Run the Collator
Now that we have node keys ready, it's time to the collator with the provided chainspec to sync our node with network. Depending on the chain size (both relay and para chains), this will take some time until the node is completely synced with the network:

```bash
./target/release/parachain-node-template --collator \
--name collator1 \
--base-path /tmp/parachain/collator1 \
--chain parachain-chainspec-raw.json \
--collator \ 
--rpc-port 10001 \
--ws-port 10002 \
--listen-addr /ip4/0.0.0.0/tcp/10003/ws \
-- \
--chain /{PATH_TO_RELAYCHAIN_CHAINSPEC} \
--port 10004 \
--ws-port 10005
```

Before jumping to the next steps, you have to wait until your node is fully synchronized:
```bash
journalctl -f -u shibuya-node -n100
```

## 3- Bind Your Account with Session Keys
Session keys are a set of cryptographic keys used by collators/validators to securely participate in the consensus mechanism and perform various roles within the network. These keys are essential for ensuring the integrity and security of the blockchain.

As a collator, you need to link your session keys to your collator account (The account which is created in step 1 and hold the funds). Once linked, the keys are used to identify your collator node. Your collator address will receive the permit to build blocks, but the session keys pass this permit to your node. 

**Author Session Keys**
In order to obtain the session keys, SSH to your node and run the below `curl` command:
```bash
curl -H "Content-Type: application/json" -d '{"id":1, "jsonrpc":"2.0", "method": "author_rotateKeys", "params":[]}' http://localhost:9944
```

The result will look like this (you just need to copy the key):
```bash
{"jsonrpc":"2.0","result":"0x600e6cea49bdeaab301e9e03215c0bcebab3cafa608fe3b8fb6b07a820386048","id":1}
```
You can access to the private key generated in this process by reaching the file generated under `chains/<chain-name>/keystore`.

**Set Session Keys**
Go to the Polkadot.js portal and connect to respective network (e.g. Rococo-AssetHub).
Go to Developper > Extrinsic and select your collator account and extrinsic type: session / setKeys
In the `Keys` section paste your session key obtained from the curl command and in the `proof` section enter `0x`.  Finally submit the call.

Note: The proof depends on the session key. For instance if your session key was started with `00x` the proof would be `00x`.


## 4- Bond Funds
Dependig on the parachain, you need a minimum amount of funds to be able to start authoring the blocks.
Go to the Polkadot.js portal: Developer > Extrinsics
Select your collator account and extrinsic type: CollatorSelection / registerAsCandidate
and finally submit the transaction.


**Congratulations!**
If you have followed all of these steps, and been selected to be a part of the collator set, you are now running a collator!
Once your collator is active, you will see your name inside Network tab every time you produce a block:



# Kubernetes
If you wish to run the collator on kubernetes environment, there's already a node helm-chart developed by parity that supports collators.

#### Add Node Helm Chart Repo
```bash
helm repo add parity https://paritytech.github.io/helm-charts/
helm -n NAMESPACE upgrade --install RELEASE_NAME -f FILE_NAME.yml parity/node
```

#### Install the node helm chart
Here's an example values file that will be needed to properly deploy the parachain:
```bash
node:
  image:
    repository: parity/polkadot-parachain
    tag: 1.14.0 
  node:
    chain: asset-hub-polkadot
    command: polkadot-parachain
    replicas: 2
    role: collator
    chainData:
      database: paritydb
      volumeSize: 100Gi
      storageClass: ssd-csi
      pruning: 1000
    chainKeystore:
      mountInMemory:
        enabled: true
    isParachain: true
    collatorRelayChain:
      chain: polkadot
      chainData:
        database: paritydb
        pruning: 256
        storageClass: ssd-csi
        volumeSize: 200Gi
        chainSnapshot:
          enabled: true
          method: http-filelist
          url: https://snapshots.polkadot.io/polkadot-paritydb-prune
        chainPath: polkadot
      chainKeystore:
        mountInMemory:
          enabled: true
    perNodeServices:
      relayP2pService:
        enabled: true
      paraP2pService:
        enabled: true
        port: 30333
      apiService:
        enabled: true
        type: ClusterIP
      setPublicAddressToExternalIp:
        enabled: true
```
