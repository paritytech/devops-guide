Kubernetes
==========


As previously mentioned, kubernetes deployments are only recommended for people with good prior operating experience of kubernetes. The management of substrate/polkadot based nodes in kubernetes is made very easy with [helm](https://helm.sh/).

Parity maintain a helm github repo @ [https://github.com/paritytech/helm-charts](https://github.com/paritytech/helm-charts) - Inside this repo is the [node](https://github.com/paritytech/helm-charts/tree/main/charts/node) chart which should be used for deploying your substate/polkadot binary. All variables are documented clearly in the [README.md](https://github.com/paritytech/helm-charts/blob/main/charts/node/README.md) and there’s an example [values.yml](https://github.com/paritytech/helm-charts/blob/main/charts/node/values.yaml) which you can start working from.

Happy Helm’ing!