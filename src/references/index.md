# Infrastructure Tooling References

This section is for listing some useful projects and tools that are relevant for node operators and developers. A community maintained [Awesome Substrate](https://github.com/substrate-developer-hub/awesome-substrate) is a more detailed general list.

## Deployment

| Project                                                                     | Description                              |
|-----------------------------------------------------------------------------|------------------------------------------|
| [Parity Helm Charts collection](https://github.com/paritytech/helm-charts)  | Parity & Polkadot Helm charts collection |
| [Polkadot Ansible collection](https://github.com/paritytech/ansible-galaxy) | Polkadot Ansible Collection              |


## Tooling

For an up-to-date list of Polkadot related tooling, check the [Polkadot Wiki Tool Index](https://wiki.polkadot.network/docs/build-tools-index).

## Indexing Chain Data

| Project                                                                  | Description                                                                                                                                                                                                                  |
| ------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [Substrate archive](https://github.com/paritytech/substrate-archive.git) | Blockchain indexing engine using PostgreSQL                                                                                                                                                                                  |
| [Subsquid](https://github.com/subsquid/squid)                            | Blockchain indexing engine using GraphQL                                                                                                                                                                                     |
| [SubQuery](https://academy.subquery.network/)                            | Open source GraphQL indexing of any Substrate network (including EVM and WASM smart contract data) directly from the RPC node. SubQuery does not require any preprocessed data making it suitable for fast changing testnets |

## CLI Utilities

| Project                                                                        | Description                                                                          |
| ------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------ |
| [Subkey](https://github.com/paritytech/substrate/tree/master/bin/utils/subkey) | Used for generating and restoring keys. Works with multiple networks and key schemes |
| [polkadot-js-tools](https://github.com/polkadot-js/tools)                      | A very flexible utility for executing API calls or running queries.                  |
| [Subxt](https://github.com/paritytech/subxt)                                   | Submits extrinsics via RPC                                                           |

## Monitoring

| Project                                                                        | Description                                                                          |
| ------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------ |
| [Polkabot](https://gitlab.com/Polkabot/polkabot)                                         | Polkabot is a Matrix chatbot that keeps an eye on the Polkadot network. |
| [polkadot-watcher-transaction](https://github.com/w3f/polkadot-watcher-transaction)      | The main use case of this application consits of a scanner that can be configured to start from a configured block number, and then it keeps monitoring the on-chain situation delivering alerts to a notifier.  |
| [polkadot-watcher-validator](https://github.com/w3f/polkadot-watcher-validator)          | The watcher is a nodeJs application meant to be connected with a substrate based node through a web socket. It can then monitor the status of the node, leveraging on mechanisms such as the builtin heartbeat. |
| [polkadot-k8s-payouts](https://github.com/w3f/polkadot-k8s-payouts)                      |  Tool that automatically claims your Kusama/Polkadot validator rewards. |

