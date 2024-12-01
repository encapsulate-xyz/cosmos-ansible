# Ansible Playbook to deploy Cosmos SDK Chains

This Ansible playbook automates the deployment and configuration of Cosmos SDK Chains. It ensures that the necessary dependencies, configuration files, and services are properly set up and running.

## Table of Contents

- [Requirements](#requirements)
- [Prerequisites](#prerequisites)
- [Setup](#setup)
- [Variables](#variables)
- [Usage](#usage)

## Requirements

Before using this playbook, ensure the following requirements are met:

1. **Ansible version**: Make sure you have Ansible 2.15+ installed.
2. **SSH Access**: Passwordless SSH access to all target servers.
3. **Python**: Python 3.x installed on the control node and all target hosts.
4. **Privileges**: The user running the playbook must have sudo privileges on the target machines.

## Prerequisites

**Install HashiCorp Vault**

This playbook relies on HashiCorp Vault to securely retrieve sensitive files, such as validator and node keys. Follow the [HashiCorp Vault Installation Guide](https://developer.hashicorp.com/vault/tutorials/getting-started/getting-started-install) to set up Vault on your infrastructure.

**Note on Secrets Management**

The playbook dynamically retrieves private validator keys and node keys from HashiCorp Vault. The keys are expected to follow a structured path format:
`<environment>/<project>/<organization>/<type>/<file_name>`
For example:
`testnet/axelar/encapsulate/validator/priv_validator_key.json`

This structure ensures easy organization and secure retrieval of secrets.

## Setup

### 1. Install Ansible

If Ansible is not installed, visit the official documentation for detailed instructions on how to install Ansible on various Linux distributions:

[Ansible Installation Guide](https://docs.ansible.com/ansible/latest/installation_guide/installation_distros.html)


### 2. Clone the repository

Clone this repository to your Ansible control node:

```bash
git clone https://github.com/encapsulate-xyz/cosmos-ansible.git
cd cosmos-ansible
```

### 3. Inventory

Define your target servers' IP address or DNS in the inventory folder, and select either `mainnet.yml` or `testnet.yml` to update.

Example for testnet.yml

```yaml
---
all:
  vars:
    env: testnet
  children:
    umee:
      hosts:
        validator.umee.testnet.encapsulate.xyz:
          type: validator
        fullnode.umee.testnet.encapsulate.xyz:
          type: fullnode
```

## Variables

This playbook allows customization through several variables. You can define these variables in the following locations:

- **`group_vars/all.yml`**: Contains all the source urls and configurations.
- **`group_vars/ports.yml`**: Contains port configurations.
- **`group_vars/mainnet.yml`** or **`group_vars/testnet.yml`**: Contains version-specific variables.
- **`group_vars/vault.yml`**: Store secret variables, such as `jwtsecret`, in this file.
- There are role specific variables defined in each roles `default/main.yml` and `vars/main.yml`.

**Note**: Make sure to set the `VAULT_TOKEN` environment variable, as it enables logging in and fetching secrets from HashiCorp Vault.

Create a `group_vars/vault.yml` with your preferred ansible-vault password:

```
ansible-vault create group_vars/vault.yml
```

Example `group_vars/vault.yml`:
```
vault:
  default:
    hcp:
      vault:
        url: https://vault.hashicorp.encapsulate.xyz
  mainnet:
  testnet:
    axelar:
      validator:
        axelar_bridge_evm:
          ethereum_sepolia_rpc:
          avalanche_rpc:
          fantom_rpc:
          moonbeam_rpc:
          polygon_rpc:
          binance_rpc:
          aurora_rpc:
          kava_rpc:
          hero_rpc:
          arbitrum_rpc:
          celo_rpc:
          optimism_rpc:          
          base_rpc:
          filecoin_rpc:
          linea_sepolia_rpc:
          immutable_rpc:
      fullnode:
        axelar_bridge_evm:
          ethereum_sepolia_rpc:
          avalanche_rpc:
          fantom_rpc:
          moonbeam_rpc:
          polygon_rpc:
          binance_rpc:
          aurora_rpc:
          kava_rpc:
          hero_rpc:
          arbitrum_rpc:
          celo_rpc: 
          optimism_rpc: 
          base_rpc:
          filecoin_rpc: 
          linea_sepolia_rpc: 
          immutable_rpc: 
```

### Usage

1. First, install the dependencies:

  ```
  ansible-galaxy role install -r roles/requirements.yml
  ansible-galaxy collection install -r collections/requirements.yml
  ```

2. Create a `ansible_vault_password` file containing ansible-vault password

3. Configure your remote server username and private key file path in the `ansible.cfg` file. Additionally, set the SSH port for your server by adjusting the `ansible_port` variable in `group_vars/all.yml`.

4. Then run the playbook:

  ```bash
  ansible-playbook node.yml -l validator.umee.testnet.encapsulate.xyz -e "fetch_validator_keys=true"
  ```

**Note**: The default value for `fetch_validator_keys` is false, which disables fetching keys from Hashicorp Vault.

After you run the playbook, it will ask for confirmation, displaying all the variables and the IP address or DNS of the server you are going to deploy.

Example output:

```bash
TASK [Display environment being deployed to] ***************************************************************************************************
ok: [validator.umee.testnet.encapsulate.xyz] => {
    "msg": [
        "Deploying to Host: validator.umee.testnet.encapsulate.xyz",
        "Groups: ['umee']",
        "Project: umee",
        "Environment: testnet",
        "Type: validator",
        "Version: v6.7.0",
        "Username: umee",
        "Service Name: umee",
        "Operating System: linux",
        "System Architecture: amd64"
    ]
}

TASK [Set all port variables dynamically] ******************************************************************************************************
ok: [validator.umee.testnet.encapsulate.xyz]

TASK [Display all port variables] **************************************************************************************************************
ok: [validator.umee.testnet.encapsulate.xyz] => {
    "msg": [
        "P2P Port: 11056",
        "RPC Port: 11057",
        "ABCI Port: 11058",
        "Prometheus Port: 11060",
        "Pprof Port: 11061",
        "REST API Port: 11017",
        "Rosetta Port: 11080",
        "gRPC Port: 11090",
        "gRPC-Web Port: 11091"
    ]
}

TASK [Confirm deployment details] ********************************************************************************************************************
Pausing for 40 seconds
(ctrl+C then 'C' = continue early, ctrl+C then 'A' = abort)
ok: [validator.umee.testnet.encapsulate.xyz]

TASK [Please confirm again] ********************************************************************************************************************
ok: [validator.umee.testnet.encapsulate.xyz] => {
    "msg": [
        "Deploying to Host: validator.umee.testnet.encapsulate.xyz",
        "Project: umee",
        "Environment: testnet",
        "Type: validator"
    ]
}

TASK [Confirm deployment details] **************************************************************************************************************
Pausing for 20 seconds
(ctrl+C then 'C' = continue early, ctrl+C then 'A' = abort)
ok: [validator.umee.testnet.encapsulate.xyz]
```
