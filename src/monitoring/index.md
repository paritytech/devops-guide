# Monitoring Overview

We can generally split monitoring into two different areas:
• On-chain monitoring - monitoring events on-chain, e.g. a transaction by a certain address, a validator set change etc…
• On-host monitoring - this is monitoring the individual node, e.g. the current block height, amount of p2p connections, amount of free memory on the host itself.

## On-Chain

This type of monitoring will generally check onchain via RPC nodes to check the values / delays / timing of events. You would normally only need two of these instances for all hosts. It is a good idea to run your own RPC servers to service these in case there are issues with the public RPC nodes.
Some example applications that queries onchain information are [polkabot](https://gitlab.com/Polkabot/polkabot) and [polkadot-basic-notification](https://github.com/paritytech/polkadot-basic-notification).

## On-Host

You should monitor each node that you run on the network. Polkadot/Substrate exposes a bunch of useful metrics on <http://host:9615/metrics>, with <http://host:9615/> being a healthcheck. This endpoint will only be exposed on the local network interface by default but you can expose it on all interfaces with the `--prometheus-external` flag.

This outputs in a simple key - value format. However you can also include tags within the key.

Simple Value:

```bash
polkadot_database_cache_bytes 0
```

Values with tags:

```bash
susbtrate_block_height{status="best"} 136
susbtrate_block_height{status="finalized"} 133
```

As the metrics provided by this endpoints don't include hosts metrics (e.g. cpu, memory, bandwidth usage), it is recommended to complement it with the [prometheus node exporter](https://github.com/prometheus/node_exporter) which needs to be installed on the same host.

## Telemetry

The telemetry server is used for real time information from nodes, showing information about their name, location, current best & finalized blocks etc… This gives you a useful dashboard to view the state of nodes.

The project is in the [substrate-telemetry](https://github.com/paritytech/substrate-telemetry) github repo, a [helm chart](https://github.com/paritytech/helm-charts/tree/main/charts/substrate-telemetry) is also available to allow easy kubernetes deployments.
