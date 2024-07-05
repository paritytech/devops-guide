## On-Host Configuration
The polkadot-sdk binary supports Prometheus metrics, which are exposed at <http://host:9615/metrics>. By default, this endpoint is only available on the local network interface. To expose it on all interfaces, you can use the --prometheus-external flag. However, when running a polkadot-parachain binary, managing the metrics becomes more complex due to two ports being used for different purposes.

#### Running the Polkadot Binary (Relay Chain)
To run the Polkadot binary with Prometheus metrics exposed, use the following command:

```
./polkadot --tmp --prometheus-external
```

#### Running the Polkadot-Parachain Binary (Parachain)
For the Polkadot-Parachain binary, use the following command to expose Prometheus metrics on a specified port:

```
./polkadot-parachain --tmp --prometheus-external --prometheus-port 9625 -- --tmp --prometheus-port 9615
```

As you can see above, running a polkadot-parachain binary requires to expose two different Prometheus port to ensure separation of metrics between relay chain and parachain.

#### Differences in Metric Labels
There are key differences in the labels of metrics exposed by the relay chain and parachain binaries:

*Relay Chain Metrics at Port 9615*
```
# TYPE polkadot_memory_allocated gauge
polkadot_memory_allocated{chain="rococo_local_testnet"} 225670544

# HELP polkadot_memory_resident Bytes allocated by the node, and held in RAM
# TYPE polkadot_memory_resident gauge
polkadot_memory_resident{chain="rococo_local_testnet"} 679718912

# HELP polkadot_node_is_active_validator Tracks if the validator is in the active set. Updates at session boundary.
# TYPE polkadot_node_is_active_validator gauge
polkadot_node_is_active_validator{chain="rococo_local_testnet"} 0
```

*Parachain Metrics at Port 9625*
```
# HELP substrate_block_height Block height info of the chain
# TYPE substrate_block_height gauge
substrate_block_height{status="best",chain="local_testnet"} 0
substrate_block_height{status="finalized",chain="local_testnet"} 0
substrate_block_height{status="sync_target",chain="local_testnet"} 0
```


#### Best Practices
To avoid confusion caused by running metrics on different ports, it is recommended to maintain uniformity by using the same Prometheus port (default is 9615) for all relay chain metrics. The chain label in the metrics helps determine which metric belongs to which chain, ensuring clarity and organization.

By following this guide, you can effectively expose and manage Prometheus metrics for both the Polkadot relay chain and parachain binaries.