Deployment Options
==================

## Node Types

A non exhaustive list of some common node types:

| Type | Function |
| ---------- | ------------------------------------------- |
| Validator | Secures the Relay Chain by staking DOT, validating proofs from collators on parachains and voting on consensus along with other validators. |
| Collator | Maintains a parachain by collecting parachain transactions and producing state transition proofs for the validators. |
| Bootnode | A node with a static address and p2p public key, used to bootstrap a node onto the network’s distributed hash table and find peers. |
| RPC Node | Expose an RPC interface over http or websocket for the relay chain or parachain and allow users to read the blockchain state and submit transactions (extrinsics). There are often multiple RPC nodes behind a load balancer. |
| Archive Node | A node which is syncing to the relay chain or parachain and has a complete database starting all the way from the genesis block. |
| Full Node | A node which is syncing the relay chain or parachain to the current best block. |


## Deployment Targets

A non exhaustive list of some common deployment targets:

| Type | Description |
| ---------- | ------------------------------------------- |
| Physical Server | Usually a rack mounted server sitting in a data center. |
| Virtual Machine | A virtual machine hosted by one of many cloud providers or self hosted onsite. |
| Kubernetes | A container orchestration engine to host your blockchain instances. This option is only recommended if you already have prior experience with kubernetes, especially in production environments. |
| Local Container | An instance running on a local container engine (e.g. containerd, docker, podman). |


## Special Options Per Host Type

| Type | Function |
| ---------- | ------------------------------------------- |
| Validator | Should at least be running with the –validator flag to enable block validation. Also keys should have been injected or the RPC author_rotateKeys called. |
| Collator | Should at least be running with the –collator flag to enable parachain collation. |
| Bootnode | A node with a static key file so the p2p public address is always the same. Store this private key in a file and use the option: --node-key-file /path/to/file |
| RPC Node | Use these options to allow 5000 public RPC/WS connections: --unsafe-ws-external --rpc-methods Safe --rpc-cors ‘*’  --ws-max-connections 5000 |
| Archive Node | Use –pruning=archive to not prune any block history |
| Full Node | N/A |
