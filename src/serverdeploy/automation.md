# Automation

## Cloud Provisioning

There are multiple options to provision cloud resources, which can either be provider specific and some tools are more cloud agnostic.

### Popular Cloud Provider Specific Deployment Tooling

- [Amazon Cloud Formation](https://aws.amazon.com/cloudformation/)
- [Azure Resource Manager](https://docs.microsoft.com/en-us/azure/azure-resource-manager/management/overview)
- [Google Cloud Deployment](https://cloud.google.com/deployment-manager/docs)

These vendor specific deployment tools are easy to use, have great examples and support. However when using multiple providers with custom scripting formats, making a basic to your infrastructure can require changes to multiple sections of code, for each provider to do the same thing (e.g. open a port on a host).

A more general solution where a change can be specified once and be used with multiple providers is using Terraform. Which currently has [more than](https://registry.terraform.io/browse/providers) 2000 providers supported.

## Terraform

[Terraform](https://www.terraform.io/) is a very commonly used tool for creating all sorts of cloud resources. It supports many different providers along with having excellent modules for the main three providers (AWS, Azure, GCP). Many projects rely on this for their infrastructure deployments.

It uses HashiCorp’s [HashiCorp Configuration Language](https://www.terraform.io/language), which makes it very easy to abstract configuration, avoid the reuse of code and enables you to plug modules together easily (e.g. RPC nodes along with a frontend load balancer).

The code is also easy to read and should be in version control. Multiple environments can be defined so you can ensure you are using the same code on dev / test / production.

Two ways to preconfigure your host after deployment using terraform directly is to either:

- Use a pre-packaged image with all of your software (e.g. packer or similar tooling)
- Use a cloud-init script to execute a certain script to preconfigure your base image

Some examples of using terraform with multiple providers can be found in the W3F’s polkadot-validator-setup github repo.

Click here for a list of [terraform providers](https://registry.terraform.io/browse/providers).

### Host Maintenance

Once your hosts are deployed you will need to configure the hosts and install the required software, configuration files etc…As mentioned this can be done in a few ways using terraform itself, however another very flexible way of configuring hosts is using ansible.

| Component | Description                                                                                                                                                                               |
| --------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Playbook  | Playbooks are the language by which Ansible orchestrates, configures, administers, or deploys systems                                                                                     |
| Role      | Roles are units of organization in Ansible. Assigning a role to a group of hosts (or a set of groups, or host patterns, and so on) implies that they should implement a specific behaviour |

When deploying our blockchain nodes, we will need an inventory which contains a list of our hosts and the groups they are in (e.g. validator, collator, rpc) and maybe some secrets which can be encrypted inline within the inventory using [ansible vault](https://docs.ansible.com/ansible/latest/user_guide/vault.html). We will then call a playbook which links hosts/groups in the inventory with roles to execute on each host.

Some examples of using ansible can be found in the paritytech's [ansible-galaxy](https://github.com/paritytech/ansible-galaxy) github repo. Specifically the node role.
