# Introduction
Collators are members of the network that maintain the parachains they take part in. They run a full node (for both their particular parachain and the relay chain), and they produce the state transition proof for relay chain validators.

There are some requirements that need to be considered prior to becoming a collator candidate including machine, bonding, account, and community requirements.
For more info regarding collators, reference [collator section](https://wiki.polkadot.network/docs/learn-collator) on Polkadot wiki page.

# prerequisites
Make sure your instance has at least the minimum recommended harware requirements:
- CPU: 2 Cores - Intel Xeon E-2386/2388 or Ryzen 9 5950x/5900x
- Memory: 8GB 
- Disk: Depending on the chain size - 200GB SSD/nvme

# Launch the Collator 

## Local Deployment

```bash
./target/release/parachain-template-node --collator \
--name <collator-name> \
--base-path /var/parachain/<collator-name> \
--chain parachain-chainspec-raw.json \
--force-authoring \
--rpc-port 10001 \
--ws-port 10002 \
--listen-addr /ip4/0.0.0.0/tcp/10003/ws \
-- \
--execution wasm \
--chain /{PATH_TO_RELAYCHAIN_CHAINSPEC} \
--port 10004 \
--ws-port 10005
```

## Kubernetes
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
