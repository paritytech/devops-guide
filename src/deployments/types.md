# Node Types

A non exhaustive list of some common node types:

| Type         | Function                                                                                                                                                                                                                      |
| ------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Validator    | Secures the Relay Chain by staking DOT, validating proofs from collators on parachains and voting on consensus along with other validators.                                                                                   |
| Collator     | Maintains a parachain by collecting parachain transactions and producing state transition proofs for the validators.                                                                                                          |
| Bootnode     | A node with a static address and p2p public key, used to bootstrap a node onto the network’s distributed hash table and find peers.                                                                                           |
| RPC Node     | Expose an RPC interface over http or websocket for the relay chain or parachain and allow users to read the blockchain state and submit transactions (extrinsics). There are often multiple RPC nodes behind a load balancer. |
| Archive Node | A node which is syncing to the relay chain or parachain and has a complete database starting all the way from the genesis block.                                                                                              |
| Full Node    | A node which is syncing the relay chain or parachain to the current best block.                                                                                                                                               |
