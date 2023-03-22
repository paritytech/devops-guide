# Helmfile

Below are two examples of helmfile in action. One a simple single helmfile for testing and the second a more realistic example to be used in production.

### Basic helmfile example

This is a very simple one file example of a helmfile to deploy two `rococo-local` relaychain validators along with two parachains `statemint-dev` and `contracts-rococo-dev`. It is intended for basic testing and familiarisation of helmfile.

A more more real world example is listed below which comes from the `testnet-manager` repo. An nginx contrainer will also be deployed to host chain spec files.

Steps:

- Create a helmfile based on the [simple example](#basic_helmfile) below and name it helmfile.yaml

- Install a webserver to host helmfiles using `kubectl create -f https://raw.githubusercontent.com/paritytech/testnet-manager/main/local-kubernetes/kube-setup/validators-chainspec.yml`

- Add relaychain and parachains using: `helmfile sync`

### Real world helmfile example

You can find a good reference helmfile example at the testnet-manager [examples](https://github.com/paritytech/testnet-manager/tree/main/local-kubernetes/charts) folder.

### Basic_helmfile

```
repositories:
  - name: parity
    url: https://paritytech.github.io/helm-charts/

helmDefaults:
  createNamespace: false
  waitForJobs: true

releases:
## relay chain Rococo ##
  - name: validator-alice
    namespace: rococo
    chart: parity/node
    version: 2.11.4
    values:
      - node:
          chain: rococo-local
          replicas: 1
          customChainspecUrl: "http://chainspec.rococo/rococo-local.json"
          role: authority
          flags:
            - "--alice"
            - "--unsafe-rpc-external"
            - "--unsafe-ws-external"
            - "--rpc-methods=unsafe"
            - "--rpc-cors=all"
            - "--node-key 0x91cb59d86820419075b08e3043cd802ba3506388d8b161d2d4acd203af5194c1"
        storageClass: standard
        extraLabels:
          validatorAccount: "5GNJqTPyNqANBkUVMN1LPPrxXnFouWXoe2wNSmmEoLctxiZY" # Alice address
          ss58Format: "0"
  - name: validator-bob
    namespace: rococo
    chart: parity/node
    version: 2.11.4
    values:
      - node:
          chain: rococo-local
          customChainspecUrl: "http://chainspec.rococo/rococo-local.json"
          role: authority
          replicas: 1
          flags:
            - "--bob"
            - "--unsafe-rpc-external"
            - "--unsafe-ws-external"
            - "--rpc-methods=unsafe"
            - "--rpc-cors=all"
        storageClass: standard
        extraLabels:
          validatorAccount: "5HpG9w8EBLe5XCrbczpwq5TSXvedjrBGCwqxK1iQ7qUsSWFc" # Bob address
          ss58Format: "0"

## Para chain Statemint ##
  - name: statemint-alice
    namespace: rococo
    chart: parity/node
    version: 2.11.4
    values:
      - image:
          repository: "parity/polkadot-parachain"
          tag: "latest"
      - node:
          chain: statemint-dev
          role: collator
          command: "/usr/local/bin/polkadot-parachain"
          collator:
            isParachain: true
            relayChainFlags: "--execution wasm --bootnodes /dns4/validator-alice-node-0/tcp/30333/p2p/12D3KooWMeR4iQLRBNq87ViDf9W7f6cc9ydAPJgmq48rAH116WoC"
            relayChainCustomChainspecUrl: "http://chainspec.rococo/rococo-local.json"
          flags:
            - "--alice"
            - "--unsafe-rpc-external"
            - "--unsafe-ws-external"
            - "--rpc-methods=unsafe"
            - "--rpc-cors=all"
            - "--force-authoring"
            - "--pruning=archive"
        storageClass: standard
        extraLabels:
          paraId: "1000"
          ss58Format: "0"

## Para chain contracts rococo
  - name: contracts-alice
    namespace: rococo
    chart: parity/node
    version: 2.11.4
    values:
      - image:
          repository: "parity/polkadot-parachain"
          tag: "latest"
      - node:
          chain: contracts-rococo-dev
          role: collator
          command: "/usr/local/bin/polkadot-parachain"
          collator:
            isParachain: true
            relayChainFlags: "--execution wasm --bootnodes /dns4/validator-alice-node-0/tcp/30333/p2p/12D3KooWMeR4iQLRBNq87ViDf9W7f6cc9ydAPJgmq48rAH116WoC"
            relayChainCustomChainspecUrl: "http://chainspec.rococo/rococo-local.json"
          flags:
            - "--alice"
            - "--unsafe-rpc-external"
            - "--unsafe-ws-external"
            - "--rpc-methods=unsafe"
            - "--rpc-cors=all"
            - "--force-authoring"
            - "--pruning=archive"
        storageClass: standard
        extraLabels:
          paraId: "1002"
          ss58Format: "0"
```
