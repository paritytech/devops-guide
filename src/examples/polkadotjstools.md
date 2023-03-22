# Polkadot Js Tools

[Polkadot Js Tools](https://github.com/polkadot-js/tools) is an amazing tool for interacting with substrate based chains.

A full list of possible [rpc](https://polkadot.js.org/docs/substrate/rpc/) calls and [extrinsics](https://polkadot.js.org/docs/substrate/extrinsics) are available from the [polkadot js docs](https://polkadot.js.org/docs/substrate/) page.

## Functions

- [api-cli](https://github.com/polkadot-js/tools/tree/master/packages/api-cli) A cli tool to allow you to make API calls to any running node

- [json-serve](https://github.com/polkadot-js/tools/blob/master/packages/json-serve) A server that serves JSON outputs for specific queries

- [monitor-rpc](https://github.com/polkadot-js/tools/blob/master/packages/monitor-rpc) A simple monitoring interface that checks the health of a remote node via RPC

- [signer-cli](https://github.com/polkadot-js/tools/blob/master/packages/signer-cli) A cli tool that allows you to generate transactions in one terminal and sign them in another terminal (or computer)

- [vanitygen](https://github.com/polkadot-js/tools/blob/master/packages/vanitygen) Generate vanity addresses, matching some pattern

### Install

Steps:

- Clone repo: `git clone https://github.com/polkadot-js/tools.git`

- Install dependencies and build: `yarn install && yarn build`

### Example Chain Queries

Display block information (parent hash, block number, state root etc..) for last block:

```
yarn run:api query.chain.getHeader
```

Get the balance for the account `//Alice`:

```
yarn run:api query.system.account 15oF4uVJwmo4TdGW7VfQxNLavjCXviqxT9S1MgbjMNHr6Sp5
```

Display all RPC methods available:

```
yarn run:api rpc.rpc.methods
```

### Example Chain Transactions/Extrinsics

Simple transfer of 1 unit `//Alice` to `//Bob` on the same chain. Using `--seed` as the sender for signing the transactions:

```
yarn run:api tx.balances.transfer 5FHneW46xGXgs5mUiveU4sbTyGBzmstUspZC92UhjJM694ty 10000000000 --seed "//Alice"
```

Transfer 1.23 units from `//Alice` on the relay chain to `//Charlie` on the parachain `1000`:

```
yarn run:api tx.xcmPallet.limitedTeleportAssets '{"v1":{"parents":0,"interior":{"x1":{"parachain":1000}}}}' '{"v1":{"parents":0,"interior":{"x1":{"AccountId32": {"id": "0x90b5ab205c6974c9ea841be688864633dc9ca8a357843eeacf2314649965fe22", "network": "Any"}}}}}' '{"v1": [ {"id": { "Concrete": {"parents":0, "interior":"Here" }}, "Fun": { "Fungible": "12300000000"}}]}'  0 Unlimited  --seed "//Alice"
```
