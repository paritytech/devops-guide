## Special Options Per Host Type

| Type | Function |
| ---------- | ------------------------------------------- |
| Validator | Should at least be running with the `--validator` flag to enable block validation. Also keys should have been injected or the RPC `author_rotateKeys` called. |
| Collator | Should at least be running with the `--collator` flag to enable parachain collation. |
| Bootnode | A node with a static key file so the p2p public address is always the same. Store this private key in a file and use the option: `--node-key-file /path/to/file` |
| RPC Node | Use these options to allow 5000 public RPC/WS connections: `--unsafe-ws-external --rpc-methods Safe --rpc-cors ‘*’  --ws-max-connections 5000` |
| Archive Node | Use `-–pruning=archive` to not prune any block history |
| Full Node | N/A |

## Parachain Specifics

When running a parachain then you will need two sets of arguments, one for the relay chain and one for the parachain. Used in the format:

```
./statemine $PARACHAIN -- $RELAYCHAIN_OPTIONS
```

A real life example of this while executing as a statemine collator would be:

```
./statemine --chain statemine --in-peers 25 --out-peers 25 --db-cache 512 --pruning=1000 --unsafe-pruning -- --chain kusama -db-cache 512 --pruning=1000 --wasm-execution Compiled
```
