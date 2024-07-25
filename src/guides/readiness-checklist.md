# Parachain Production Readiness Checklist

Before launching your parachain network, verify the readiness of your network infrastructure by reviewing the following five key points.

<!-- toc -->

  * [1. Launch your parachain on a testnet before launching on mainnet](#1-launch-your-parachain-on-a-testnet-before-launching-on-mainnet)
  * [2. Use Infrastructure as Code](#2-use-infrastructure-as-code)
  * [3. Ensure redundancy for your network nodes](#3-ensure-redundancy-for-your-network-nodes)
  * [4. Check that your nodes are properly configured to fulfil their roles](#4-check-that-your-nodes-are-properly-configured-to-fulfil-their-roles)
    + [Chainspec checks](#chainspec-checks)
    + [Bootnode checks](#bootnode-checks)
    + [Collator checks](#collator-checks)
    + [RPC checks](#rpc-checks)
  * [5. Validate that your nodes are fully synced](#5-validate-that-your-nodes-are-fully-synced)
  * [6. Set up monitoring and alerting](#6-set-up-monitoring-and-alerting)
- [Tips:](#tips)

<!-- tocstop -->

## 1. Launch your parachain on a testnet before launching on mainnet

The best way to validate your deployment is to deploy a testnet for your parachain.
The recommended relay-chain testnet for builders is now [Paseo](https://github.com/paseo-network) where you can benefit from a stable and mainnet-like experience before onboarding to Polkadot.

## 2. Set up your nodes using Infrastructure as Code

Using Infrastructure as Code for your blockchain nodes helps with the following:

- **Consistency and Reproducibility**: Reduces manual configurations and allows configuration reuse across testnet and mainnet.
- **Automation and Efficiency**: Although it takes upfront work to set up, you will speed up further deployments by reducing the number of manual steps.
- **Disaster Recovery**: In the event of a disaster, you will be able to quickly restore your infrastructure.

## 3. Ensure redundancy for your network nodes

For a reliable network, it is recommended to have at least:
- 2 Bootnodes
- 2 Collators
- 2 RPC nodes behind a load balancer

This will help ensure the smooth operation of the network, allowing one node to restart or upgrade while the other still performs its function.
After launch, it is a good idea to increase the size of the network further and decentralize node operations to multiple individuals and organizations.

## 4. Check that your nodes are properly configured to fulfil their roles

### Chainspec checks

Your chainspec must:
- Have all the required properties such as `para_id` and `relay_chain`
- Be [converted to raw format](./parachain_deployment.md#convert-your-plain-chainspec-to-raw)
- Have the desired chain state in [its genesis block viewable in a local parachain dry-run](./parachain_deployment.md#optional-dry-run-your-parachain-network-locally)
- Reference your bootnodes addresses so your nodes will connect to the correct network on startup

### Bootnode checks

Your bootnodes must have a fixed network ID and fixed IP or DNS in their addresses, e.g., /dns/polkadot-bootnode-0.polkadot.io/tcp/30333/p2p/12D3KooWSz8r2WyCdsfWHgPyvD8GKQdJ1UAiRmrcrs8sQB3fe2KU.
This can be achieved with these flags, e.g:

```
--node-key-file <key-file> --listen-addr=/ip4/0.0.0.0/tcp/30333 --listen-addr /ip6/::/tcp/30333 --public-addr=/ip4/PUBLIC_IP/tcp/30333.
```

⚠️ Your bootnodes should always keep the same addresses across restarts.

### Collator checks

- Each of your collators must have an aura key in their keystore, as explained in the [collator guide](TODO link to collator guide)
- The aura keys also need to be set on-chain, either:
  * in the [genesis state directly in the chainspec file](./parachain_deployment.html#prepare-your-genesis-patch-config)
  * by [submitting a setKeys extrinsic with your collator account](TODO link to collator guide)

### RPC checks

Your RPC nodes should have those flags enabled:
- `--rpc-external` or `--unsafe-rpc-external` (does the same but doesn't output a warn log)
- `--rpc-methods Safe`
- `--rpc-cors *`
- `--pruning=archive`, you generally want to have your RPC nodes to be archives to access historical blocks.

You can also add RPC protections flags, e.g.:

- `--rpc-max-connections 1000`: Allow a maximum of 1000 simultaneous open connection
- `--rpc-rate-limit=10`: Limit to 10 calls per minute
- `--rpc-rate-limit-whitelisted-ips 10.0.0.0/8 1.2.3.4/32`: Disable RPC rate limiting for certain ip addresses
- `--rpc-rate-limit-trust-proxy-headers`: Trust proxy headers for determining the IP for rate limiting

## 5. Validate that your nodes are fully synced

Your parachain nodes should be fully synced with the latest blocks of both your parachain and the relay-chain it is connected to.

## 6. Set up monitoring and alerting

At a minimum, it is recommended to collect your nodes logs and be notified whenever an error or panic logs happens.

It is highly recommended to set up metrics based monitoring and alerting, including:
- General system-level metrics, e.g. by using the [prometheus node exporter](https://awesome-prometheus-alerts.grep.to/rules.html#host-and-hardware) to watch over disk, cpu, memory, oom kills, etc.
- [Polkadot-sdk native metrics](../monitoring/infrastructure.md#metrics)

# Tips:

* For troubleshooting testnets collators you can set`--log parachain=debug`.
Other useful debug targets are`runtime=debug`, `sync=debug`, `author=debug`, `xcm=debug`, etc.
* You should avoid setting trace logging on all your nodes; if you do, set it on a limited number of nodes where it is useful and remove it when not needed anymore.