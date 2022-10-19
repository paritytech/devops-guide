Ansible
====================

Ansible is a popular tool to detect if a host matches the current expected state and also apply any changes or updates if required. Lots of create pre-packaged modules exist in [ansible galaxy](https://galaxy.ansible.com/). 

The paritytech.chain_operations module is maintained on [github](https://github.com/paritytech/ansible-galaxy) and available via [ansible-galaxy]()


### Installation

Create a `requirements.yml` file and use `ansible-galaxy` to download the latest version. See the [chain_operations README.md](https://github.com/paritytech/ansible-galaxy/blob/main/README.md) for setup instructions.


### Deploy Development Network

Goal: Deploy two relaychain validators and one parachain collator.

It expects a relaychain and parachain chain specification. A [guide](https://docs.substrate.io/reference/how-to-guides/parachains/connect-to-a-relay-chain/) on how to create these chain specifications is available. This network will use default `Alice` and `Bob` keys.

#### Development Network Inventory:

```yaml
all:
  vars:
    node_binary_version: v0.9.27
    node_app_name: testnet-local
    node_chainspec: https://mylocation.com/rococo-local-raw-default.json
    node_parachain_chainspec: https://mylocation.com/statemint-dev.json
    node_user: polkadot
    node_binary: "https://github.com/paritytech/polkadot/releases/download/{{ node_binary_version }}/polkadot"
    onboard_para_sudo_key: "0xe5be9a5092b81bca64be81d212e7f2f9eba183bb7a90954f7b76361f6edb5c0a"
    onboard_para_url: "ws://127.0.0.1:9944"
    onboard_para_wait: true
    onboard_para_parachain_state: "present"

  children:
    validators:
      hosts:
        validator-1:
          node_custom_options: ["--alice"]
          ansible_host: validator-1.mycompany.com
          node_p2p_private_key: "0x0"
        validator-2:
          node_custom_options: ["--bob"]
          ansible_host: validator-2.mycompany.com
    collator:
      hosts:
        collator-1:
          node_parachain_role: collator
          node_custom_options: ["--execution wasm"]
          node_parachain_custom_options: ["--alice --force-authoring"]
          ansible_host: collator-1.mycompany.com
          node_binary: "https://github.com/paritytech/cumulus/releases/download/{{ node_binary_version }}0/polkadot-parachain"
          onboard_para_parachain_id: 1000
```


#### Development Network Deployment Playbook:

```yaml
---
- name: deploy all nodes
  hosts: all
  become: yes
  roles:
    - parity.chain.node
---
- name: onboard parachain
  hosts: collator
  become: yes
  roles:
    - parity.chain.onboard_para
```



### Deploy Custom Network

Goal: Deploy two relaychain validators and one parachain collator, this time with custom validator keys instead of using `Alice` and `Bob`. Custom keys will be given in the inventory file and injected during deployment.

A [guide](https://docs.substrate.io/tutorials/get-started/add-trusted-nodes/) is available on creating a chain spec with custom keys.


#### Custom Network Inventory:


```yaml
all:
  vars:
    node_binary_version: v0.9.27
    node_app_name: custom-local
    node_chainspec: https://mylocation.com/rococo-local-raw-default.json
    node_parachain_chainspec: https://mylocation.com/statemint-dev.json
    node_user: polkadot
    node_binary: "https://github.com/paritytech/polkadot/releases/download/{{ node_binary_version }}/polkadot"
    onboard_para_sudo_key: "0x0"
    onboard_para_url: "ws://127.0.0.1:9944"
    onboard_para_wait: true
    onboard_para_parachain_state: "present"

  children:
    validators:
      hosts:
        validator-1:
          node_custom_options: ["--validator"]
          key_inject_validator_seed: "0x0" 
          ansible_host: validator-1.mycompany.com
        validator-2:
          node_custom_options: ["--validator"]
          ansible_host: validator-2.mycompany.com
          key_inject_validator_seed: "0x0"
    collator:
      hosts:
        collator-1:
          node_parachain_role: collator
          node_custom_options: ["--execution wasm"]
          node_parachain_custom_options: ["--force-authoring"]
          ansible_host: collator-1.mycompany.com
          node_binary: "https://github.com/paritytech/cumulus/releases/download/{{ node_binary_version }}0/polkadot-parachain"
          onboard_para_parachain_id: 1000
          key_inject_parachain_aura_private_key: "0x0"
```

#### Custom Network Deployment Playbook:

```yaml
---
- name: deploy all nodes
  hosts: all
  become: yes
  roles:
    - parity.chain.node
---
- name: inject keys on relaychain validators
  hosts: validators
  become: yes
  roles:
    - parity.chain.inject_keys
---
- name: onboard parachain
  hosts: collator
  become: yes
  roles:
    - parity.chain.onboard_para
```
