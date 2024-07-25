# Custom Chain - Chainspecs

This section will explain what is required to build a custom chain spec for your own network.
For more in depth explanations on how chainspecs works internally, refer to the [Polkadot-sdk Chainspec docs](https://paritytech.github.io/polkadot-sdk/master/polkadot_sdk_docs/reference_docs/chain_spec_genesis/index.html)

## Create Chain Spec with the node binary

You can create a plain chainspec for your network with:

```bash
node build-spec –chain chain-name --disable-default-bootnode > custom-chainspec.json
```

**Important Fields**:

| Field                  | Purpose                                                              |
|------------------------|----------------------------------------------------------------------|
| Name                   | Name of the network                                                  |
| id                     | Id of network, also used for the filesystem path                     |
| bootNodes              | List of multiaddr’s of bootnodes for the network                     |
| protocolId             | A unique identifier for the protocol                                 |
| para_id                | (Parachains only) numerical ID of the parachain slot                 |
| relay_chain            | (Parachains only) Id of the relaychain to connect to                 |
| genesis.runtimeGenesis | (Plain chainspec only) Configuration of the genesis runtime          |
| genesis.raw            | (Raw chainspec only) Serialization of the raw of the genesis storage |

Once the values have been updated this chainspec should be converted in raw format using the command:

```bash
node build-spec –chain custom-chainspec.json –raw > custom-chainspec-raw.json
```

## Using the chain-spec-builder

The chain-spec builder is a utility that can create a chainspec from a runtime wasm. The tool's linux binary can be downloaded from the [polkadot-sdk release page](https://github.com/paritytech/polkadot-sdk/releases) or built from sources in [polkadot-sdk/substrate/bin/utils/chain-spec-builder](https://github.com/paritytech/polkadot-sdk/tree/master/substrate/bin/utils/chain-spec-builder).

### Create a genesis.json file from presets

Runtimes can have config presets which are examples of valid genesis configuration files that can be edited for customizing chainspecs.

#### List the available presets of a runtime

    $ chain-spec-builder list-presets --runtime-wasm-path rococo_runtime-v1015000.compact.compressed.wasm

```json
{
  "presets": [
    "local_testnet",
    "development",
    "staging_testnet",
    "wococo_local_testnet",
    "versi_local_testnet"
  ]
}
```

Unfortunately, not all existing runtimes have embedded presets, if those are not available, you should ask your chain developers to move their genesis config presets from the node binary to the runtime (see eg. [polkadot-sdk#4749](https://github.com/paritytech/polkadot-sdk/pull/4739)).


#### Export a config preset from a runtime file

    $ chain-spec-builder display-preset --runtime-wasm-path rococo_runtime-v1015000.compact.compressed.wasm --preset-name staging_testnet > rococo-genesis.json


<details>
<summary>rococo-genesis.json</summary>
<code style="display:block; white-space:pre-wrap">
{
  "babe": {
    "epochConfig": {
      "allowed_slots": "PrimaryAndSecondaryVRFSlots",
      "c": [
        1,
        4
      ]
    }
  },
  "balances": {
    "balances": [
      [
        "5DwBmEFPXRESyEam5SsQF1zbWSCn2kCjyLW51hJHXe9vW4xs",
        1000000000000000000
      ],
      [
        "5EHZkbp22djdbuMFH9qt1DVzSCvqi3zWpj6DAYfANa828oei",
        100000000000000
      ],
      [
        "5DvH8oEjQPYhzCoQVo7WDU91qmQfLZvxe9wJcrojmJKebCmG",
        100000000000000
      ],
      [
        "5FPMzsezo1PRxYbVpJMWK7HNbR2kUxidsAAxH4BosHa4wd6S",
        100000000000000
      ],
      [
        "5DMNx7RoX6d7JQ38NEM7DWRcW2THu92LBYZEWvBRhJeqcWgR",
        100000000000000
      ],
      [
        "5C8AL1Zb4bVazgT3EgDxFgcow1L4SJjVu44XcLC9CrYqFN4N",
        100000000000000
      ],
      [
        "5C8XbDXdMNKJrZSrQURwVCxdNdk8AzG6xgLggbzuA399bBBF",
        100000000000000
      ],
      [
        "5HinEonzr8MywkqedcpsmwpxKje2jqr9miEwuzyFXEBCvVXM",
        100000000000000
      ],
      [
        "5Ey3NQ3dfabaDc16NUv7wRLsFCMDFJSqZFzKVycAsWuUC6Di",
        100000000000000
      ]
    ]
  },
  "configuration": {
    "config": {
      "approval_voting_params": {
        "max_approval_coalesce_count": 1
      },
      "async_backing_params": {
        "allowed_ancestry_len": 2,
        "max_candidate_depth": 3
      },
      "code_retention_period": 1200,
      "dispute_period": 6,
      "dispute_post_conclusion_acceptance_period": 100,
      "executor_params": [],
      "hrmp_channel_max_capacity": 8,
      "hrmp_channel_max_message_size": 1048576,
      "hrmp_channel_max_total_size": 8192,
      "hrmp_max_message_num_per_candidate": 5,
      "hrmp_max_parachain_inbound_channels": 4,
      "hrmp_max_parachain_outbound_channels": 4,
      "hrmp_recipient_deposit": 0,
      "hrmp_sender_deposit": 0,
      "max_code_size": 3145728,
      "max_downward_message_size": 1048576,
      "max_head_data_size": 32768,
      "max_pov_size": 5242880,
      "max_upward_message_num_per_candidate": 5,
      "max_upward_message_size": 51200,
      "max_upward_queue_count": 8,
      "max_upward_queue_size": 1048576,
      "max_validators": null,
      "minimum_backing_votes": 2,
      "minimum_validation_upgrade_delay": 5,
      "n_delay_tranches": 25,
      "needed_approvals": 2,
      "no_show_slots": 2,
      "node_features": {
        "bits": 8,
        "data": [
          2
        ],
        "head": {
          "index": 0,
          "width": 8
        },
        "order": "bitvec::order::Lsb0"
      },
      "pvf_voting_ttl": 2,
      "relay_vrf_modulo_samples": 2,
      "scheduler_params": {
        "group_rotation_frequency": 20,
        "lookahead": 2,
        "max_availability_timeouts": 0,
        "max_validators_per_core": null,
        "num_cores": 0,
        "on_demand_base_fee": 10000000,
        "on_demand_fee_variability": 30000000,
        "on_demand_queue_max_size": 10000,
        "on_demand_target_queue_utilization": 250000000,
        "paras_availability_period": 4,
        "ttl": 5
      },
      "validation_upgrade_cooldown": 2,
      "validation_upgrade_delay": 2,
      "zeroth_delay_tranche_width": 0
    }
  },
  "registrar": {
    "nextFreeParaId": 2000
  },
  "session": {
    "keys": [
      [
        "5EHZkbp22djdbuMFH9qt1DVzSCvqi3zWpj6DAYfANa828oei",
        "5EHZkbp22djdbuMFH9qt1DVzSCvqi3zWpj6DAYfANa828oei",
        {
          "authority_discovery": "5HbSgM72xVuscsopsdeG3sCSCYdAeM1Tay9p79N6ky6vwDGq",
          "babe": "5Fh6rDpMDhM363o1Z3Y9twtaCPfizGQWCi55BSykTQjGbP7H",
          "beefy": "KWAWRuYGuB7eAXWBsb7ZtaA267AUABJCP45vA7T61bNKG3zMc",
          "grandpa": "5CPd3zoV9Aaah4xWucuDivMHJ2nEEmpdi864nPTiyRZp4t87",
          "para_assignment": "5HQdwiDh8Qtd5dSNWajNYpwDvoyNWWA16Y43aEkCNactFc2b",
          "para_validator": "5CP6oGfwqbEfML8efqm1tCZsUgRsJztp9L8ZkEUxA16W8PPz"
        }
      ],
      [
        "5DvH8oEjQPYhzCoQVo7WDU91qmQfLZvxe9wJcrojmJKebCmG",
        "5DvH8oEjQPYhzCoQVo7WDU91qmQfLZvxe9wJcrojmJKebCmG",
        {
          "authority_discovery": "5HeXbwb5PxtcRoopPZTp5CQun38atn2UudQ8p2AxR5BzoaXw",
          "babe": "5DLjSUfqZVNAADbwYLgRvHvdzXypiV1DAEaDMjcESKTcqMoM",
          "beefy": "KWCXxi2nesB7EUaPMr5oExVcaJ5AmN5adHtWdXeqoEhmMcTK7",
          "grandpa": "5HnDVBN9mD6mXyx8oryhDbJtezwNSj1VRXgLoYCBA6uEkiao",
          "para_assignment": "5ES3fw5X4bndSgLNmtPfSbM2J1kLqApVB2CCLS4CBpM1UxUZ",
          "para_validator": "5EPEWRecy2ApL5n18n3aHyU1956zXTRqaJpzDa9DoqiggNwF"
        }
      ],
      [
        "5FPMzsezo1PRxYbVpJMWK7HNbR2kUxidsAAxH4BosHa4wd6S",
        "5FPMzsezo1PRxYbVpJMWK7HNbR2kUxidsAAxH4BosHa4wd6S",
        {
          "authority_discovery": "5D4r6YaB6F7A7nvMRHNFNF6zrR9g39bqDJFenrcaFmTCRwfa",
          "babe": "5GpZhzAVg7SAtzLvaAC777pjquPEcNy1FbNUAG2nZvhmd6eY",
          "beefy": "KWCGCQqZsnEB4jitvZstHq9dxJE4unuAga5naKmPUUs1oVya8",
          "grandpa": "5HAes2RQYPbYKbLBfKb88f4zoXv6pPA6Ke8CjN7dob3GpmSP",
          "para_assignment": "5CtK7JHv3h6UQZ44y54skxdwSVBRtuxwPE1FYm7UZVhg8rJV",
          "para_validator": "5FtAGDZYJKXkhVhAxCQrXmaP7EE2mGbBMfmKDHjfYDgq2BiU"
        }
      ],
      [
        "5DMNx7RoX6d7JQ38NEM7DWRcW2THu92LBYZEWvBRhJeqcWgR",
        "5DMNx7RoX6d7JQ38NEM7DWRcW2THu92LBYZEWvBRhJeqcWgR",
        {
          "authority_discovery": "5CtgRR74VypK4h154s369abs78hDUxZSJqcbWsfXvsjcHJNA",
          "babe": "5EjkyPCzR2SjhDZq8f7ufsw6TfkvgNRepjCRQFc4TcdXdaB1",
          "beefy": "KW8tZu7Cf6dTjGHR6SwQcoAry4LUVCTq3pzfUNSketjYc8NPC",
          "grandpa": "5DJV3zCBTJBLGNDCcdWrYxWDacSz84goGTa4pFeKVvehEBte",
          "para_assignment": "5F1FZWZSj3JyTLs8sRBxU6QWyGLSL9BMRtmSKDmVEoiKFxSP",
          "para_validator": "5F9FsRjpecP9GonktmtFL3kjqNAMKjHVFjyjRdTPa4hbQRZA"
        }
      ],
      [
        "5C8AL1Zb4bVazgT3EgDxFgcow1L4SJjVu44XcLC9CrYqFN4N",
        "5C8AL1Zb4bVazgT3EgDxFgcow1L4SJjVu44XcLC9CrYqFN4N",
        {
          "authority_discovery": "5DABsdQCDUGuhzVGWe5xXzYQ9rtrVxRygW7RXf9Tsjsw1aGJ",
          "babe": "5Et3tfbVf1ByFThNAuUq5pBssdaPPskip5yob5GNyUFojXC7",
          "beefy": "KW8bmi1qiPmp5x2PAfqBeQ8MdAGWDXo7fYeAAB92oWgXdvXmL",
          "grandpa": "5EX1JBghGbQqWohTPU6msR9qZ2nYPhK9r3RTQ2oD1K8TCxaG",
          "para_assignment": "5CaZuueRVpMATZG4hkcrgDoF4WGixuz7zu83jeBdY3bgWGaG",
          "para_validator": "5EUNaBpX9mJgcmLQHyG5Pkms6tbDiKuLbeTEJS924Js9cA1N"
        }
      ],
      [
        "5C8XbDXdMNKJrZSrQURwVCxdNdk8AzG6xgLggbzuA399bBBF",
        "5C8XbDXdMNKJrZSrQURwVCxdNdk8AzG6xgLggbzuA399bBBF",
        {
          "authority_discovery": "5CZdFnyzZvKetZTeUwj5APAYskVJe4QFiTezo5dQNsrnehGd",
          "babe": "5GHWB8ZDzegLcMW7Gdd1BS6WHVwDdStfkkE4G7KjPjZNJBtD",
          "beefy": "KW2vnN5A1F59icBUNrF3gpcTdDMb8pvYPVACctkGHkdAW3oDB",
          "grandpa": "5GzDPGbUM9uH52ZEwydasTj8edokGUJ7vEpoFWp9FE1YNuFB",
          "para_assignment": "5DnsSy8a8pfE2aFjKBDtKw7WM1V4nfE5sLzP15MNTka53GqS",
          "para_validator": "5CmLCFeSurRXXtwMmLcVo7sdJ9EqDguvJbuCYDcHkr3cpqyE"
        }
      ],
      [
        "5HinEonzr8MywkqedcpsmwpxKje2jqr9miEwuzyFXEBCvVXM",
        "5HinEonzr8MywkqedcpsmwpxKje2jqr9miEwuzyFXEBCvVXM",
        {
          "authority_discovery": "5ELv74v7QcsS6FdzvG4vL2NnYDGWmRnJUSMKYwdyJD7Xcdi7",
          "babe": "5EeCsC58XgJ1DFaoYA1WktEpP27jvwGpKdxPMFjicpLeYu96",
          "beefy": "KWA93sGFQ4a2iMR9DGQt7iQj3NZwKW8mWDZU6gZzU6jYubBMX",
          "grandpa": "5DnEySxbnppWEyN8cCLqvGjAorGdLRg2VmkY96dbJ1LHFK8N",
          "para_assignment": "5HjRTLWcQjZzN3JDvaj1UzjNSayg5ZD9ZGWMstaL7Ab2jjAa",
          "para_validator": "5CAC278tFCHAeHYqE51FTWYxHmeLcENSS1RG77EFRTvPZMJT"
        }
      ],
      [
        "5Ey3NQ3dfabaDc16NUv7wRLsFCMDFJSqZFzKVycAsWuUC6Di",
        "5Ey3NQ3dfabaDc16NUv7wRLsFCMDFJSqZFzKVycAsWuUC6Di",
        {
          "authority_discovery": "5DqAvikdpfRdk5rR35ZobZhqaC5bJXZcEuvzGtexAZP1hU3T",
          "babe": "5H168nKX2Yrfo3bxj7rkcg25326Uv3CCCnKUGK6uHdKMdPt8",
          "beefy": "KW54aj4HK5xHt3eqKqLuPRMDY8kRi4VgKycb3Uq7Bu6fQxqwA",
          "grandpa": "5DrA2fZdzmNqT5j6DXNwVxPBjDV9jhkAqvjt6Us3bQHKy3cF",
          "para_assignment": "5DhDcHqwxoes5s89AyudGMjtZXx1nEgrk5P45X88oSTR3iyx",
          "para_validator": "5Gx6YeNhynqn8qkda9QKpc9S7oDr4sBrfAu516d3sPpEt26F"
        }
      ]
    ]
  },
  "sudo": {
    "key": "5DwBmEFPXRESyEam5SsQF1zbWSCn2kCjyLW51hJHXe9vW4xs"
  }
}
</code>
</details>


### Create a chain-spec from a runtime wasm file


```bash
chain-spec-builder -c chainspec.json create -n "Test Chain" -i test-chain -t live -r runtime.wasm patch genesis.json
```


### Verify a chain-spec file


```bash
chain-spec-builder verify chainspec.json
```

### Convert a plain chain-spec to raw

```bash
chain-spec-builder convert-to-raw chainspec.json  > chainspec.raw.json
```