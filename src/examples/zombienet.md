Zombienet
====================

Below is an example that is similar to the helmfile example. It will deploy two `rococo-local` relaychain validators and two parachains `statemint-dev` and `contracts-rococo-dev` and also HRMP channels between parachains 1000 and 1002.

### More Information

For more information see the [Zombienet manual](https://paritytech.github.io/zombienet/). For more examples see the [Zomebienet examples](https://github.com/paritytech/zombienet/tree/main/examples) in the Zombienet repo.


### Steps

- Clone the zombienet repo: `git@github.com:paritytech/zombienet.git`

- Build zomebienet using: `npm install && npm run build`

- Create a new config file based on the [zombienet_example config](#zombienet_example)

- Deploy this to k8s using the command: `node cli/dist.js spawn path/configfile.yaml`



### Zombienet_example

```
relaychain]
default_image = "docker.io/paritypr/polkadot-debug:master"
default_command = "polkadot"
default_args = [ "-lparachain=debug" ]

chain = "rococo-local"

  [[relaychain.nodes]]
  name = "alice"
  validator = true

  [[relaychain.nodes]]
  name = "bob"
  validator = true

[[parachains]]
id = 1000
cumulus_based = true
chain = "statemint-dev"

  [parachains.collator]
  name = "statemint-collator"
  image = "docker.io/parity/polkadot-parachain:latest"
  command = "polkadot-parachain"
  args = ["-lparachain=debug"]

[[parachains]]
id = 1002
cumulus_based = true
chain = "contracts-rococo-dev"

  [parachains.collator]
  name = "contracts-collator"
  image = "docker.io/parity/polkadot-parachain:latest"
  command = "polkadot-parachain"
  args = ["-lparachain=debug"]

[[hrmpChannels]]
sender = 1000
recipient = 1002
maxCapacity = 4
maxMessageSize = 512

[[hrmpChannels]]
sender = 1002 
recipient = 1000 
maxCapacity = 4
maxMessageSize = 512
```
