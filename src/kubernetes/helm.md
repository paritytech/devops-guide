Helm Chart
=============


## Overview

Parity maintain a helm github repo @ [https://github.com/paritytech/helm-charts](https://github.com/paritytech/helm-charts) - Inside this repo is the [node](https://github.com/paritytech/helm-charts/tree/main/charts/node) chart which should be used for deploying your substate/polkadot binary. 


All variables are documented clearly in the [README.md](https://github.com/paritytech/helm-charts/blob/main/charts/node/README.md) and there’s an example [values.yml](https://github.com/paritytech/helm-charts/blob/main/charts/node/values.yaml) which you can start working from.


## Examples

First install the helm repo:

```
helm repo add parity https://paritytech.github.io/helm-charts/
helm install polkadot-node parity/node
```

Example Polkadot Deployment:

```
helm install polkadot-node parity/node --set node.chainDataSnapshotUrl=https://dot-rocksdb.polkashots.io/snapshot --set node.chainDataSnapshotFormat=lz4
```

Important Options:

| Option | Description |
| ---------- | ------------------------------------------- |
| node.chain | Network to connect to |
| node.command | Binary to use |
| node.flags | Flags to use with binary in container |
| node.customChainspecUrl | Custon Chainspec URL |

