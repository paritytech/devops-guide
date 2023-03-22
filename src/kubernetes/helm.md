# Helm Chart

## Overview

Parity maintain a helm github repo @ [https://github.com/paritytech/helm-charts](https://github.com/paritytech/helm-charts) - Inside this repo is the [node](https://github.com/paritytech/helm-charts/tree/main/charts/node) chart which should be used for deploying your substrate/polkadot binary.

All variables are documented clearly in the [README.md](https://github.com/paritytech/helm-charts/blob/main/charts/node/README.md) and there’s an example [values.yml](https://github.com/paritytech/helm-charts/blob/main/charts/node/values.yaml) which you can start working from.

### Example Rococo Local Chain

This is a simple example of deploying a `rococo-local` test chain in kubernetes. Two validators and two full nodes will be deployed via the helm chart. Once both validators are running you will see block production. A custom node key is used on the `Alice` validator which all other hosts use as a bootnode.

#### First install the helm repo

```bash
helm repo add parity https://paritytech.github.io/helm-charts/
helm install polkadot-node parity/node
```

#### Deploy Validator Alice

Alice will be deployed in a stateful set and use an example custom node key, along with deploying a service to be used as a bootnode:

```bash
helm install rococo-alice parity/node --set node.role="validator" --set node.customNodeKey="91cb59d86820419075b08e3043cd802ba3506388d8b161d2d4acd203af5194c1" --set node.chain=rococo-local --set node.perNodeServices.relayP2pService.enabled=true --set node.perNodeServices.relayP2pService.port=30333 --set node.flags="--alice --rpc-external --ws-external --rpc-cors all --rpc-methods=unsafe"
```

#### Deploy Validator Bob

```bash
helm install rococo-bob parity/node --set node.role="validator" --set node.chain=rococo-local --set node.flags="--bob --bootnodes '/dns4/rococo-alice-node-0-relay-chain-p2p/tcp/30333/p2p/12D3KooWMeR4iQLRBNq87ViDf9W7f6cc9ydAPJgmq48rAH116WoC'"
```

#### Deploy Two Full Nodes

```bash
helm install rococo-pool parity/node --set node.chain=rococo-local --set node.replicas=2 --set node.flags="--bootnodes '/dns4/rococo-alice-node-0-relay-chain-p2p/tcp/30333/p2p/12D3KooWMeR4iQLRBNq87ViDf9W7f6cc9ydAPJgmq48rAH116WoC'"
```

Once these steps are complete you will have a working `rococo-local` test chain with two validators and two full nodes.

#### GitOps Tooling

Below are some useful GitOps tool for managing helm releases. Here is a list of tool from simplest to more advanced:

- [Helmfile](https://github.com/roboll/helmfile)
- [Terraform Helm provider](https://registry.terraform.io/providers/hashicorp/helm/latest/docs)
- [Flux CD](https://fluxcd.io/)
- [ArgoCD](https://argo-cd.readthedocs.io/en/stable/)

## Important Chart Options

| Option                  | Description                           |
| ----------------------- | ------------------------------------- |
| node.chain              | Network to connect to                 |
| node.command            | Binary to use                         |
| node.flags              | Flags to use with binary in container |
| node.customChainspecUrl | Custom Chainspec URL                  |
