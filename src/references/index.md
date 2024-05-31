# Infrastructure Tooling References

This section is for listing some useful projects and tools that are relevant for node operators and developers. A community maintained [Awesome Substrate](https://github.com/substrate-developer-hub/awesome-substrate) is a more detailed general list.

## Testing

| Project                                                  | Description                                                                                                                                           |
| -------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- |
| [Zombienet](https://github.com/paritytech/zombienet)     | A great tool for deploying test setups. Providers include native, Podman and Kubernetes. Also supports running automated tests against these networks |
| [smart bench](https://github.com/paritytech/smart-bench) | Smart contracts benchmarking on Substrate                                                                                                             |

## Frontends

| Project                                                                                          | Description                                                                                                                          |
| ------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------ |
| [polkadot-js frontend](https://github.com/polkadot-js/apps)                                      | GitHub repo for [https://polkadot.js.org/apps/] - commonly used application for interacting with substrate and polkadot based chains |
| [Staking Dashboard](https://github.com/paritytech/polkadot-staking-dashboard)                    | A sleek [staking dashboard](https://staking.polkadot.network/dashboard) using react and the polkadot-js library                      |
| [contracts-ui](https://github.com/paritytech/contracts-ui)                                       | Web application for deploying Wasm smart contracts on Substrate chains that include the FRAME contracts pallet                       |
| [Example polkadot-js-bundle](https://github.com/polkadot-js/common/blob/master/test-bundle.html) | Use polkadot JavaScript bundles to write custom frontends                                                                            |

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
