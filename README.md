# Polkadot Validation Node Ansible Setup

This repo is to set up the Polkadot Validation node. This repo is heavily influenced by https://github.com/w3f/polkadot-validator-setup, the official Polkadot secure setup guide.

## Motivation

While the official setup is very comprehensive, it can be overwhelming for "small" validators who do not care much about using Terraform on the infrastructure layer.

## Summary

You run one playbook to prepare a node with Node Exporter and Promtail, and run one more playbook to launch a Kusama/Polkadot node. Boom!

```bash
ansible-playbook prepare.yml -e "target=VALIDATOR_TARGET"
ansible-playbook polkadot.yml -e "target=VALIDATOR_TARGET"
```

But before you rush with this easy setup, you probably want to read on so you understand the structure of this Ansible program and all the features it offers.

## Some Preparations

First of all, some preparation is in order.

Make sure that you have a production inventory file with your confidential server info. You will start by copying the sample inventory file (included in the repo). The sample file gives you a good idea on how to define the inventory.

```bash
cp inventory.sample.ini inventory.ini
```

Needless to say, you need to update the dummy values in the inventory file. For each Kusama/Polkadot node, you need to update:

1. ansible_host: Your server public IP
1. validator_name: This is the node name that will show up on telemetry monitoring board. It is especially important if you want to participate in the Thousand Validators Program. For us, we use something like `polkachu-kusama-01` and `polkachu-polkadot-02` to keep it unique and organized.
1. port_prefix: This allows you to install multiple nodes on the same server without port conflict

## Basic Cluster Structure

The basic cluster structure is:

1. Name each Kusama node as `kusama1`, `kusama2`, etc. Group all Kusama nodes into `kusama` group.
2. Name each Polkadot node as `polkadot1`, `polkadot2`, etc. Group all Polkadot nodes into `polkadot` group.

The structure allows you to target `vars` to each node, or either Kusama or Polkadot cluster, or the whole cluster.

## Main Playbook to Set Up a Kusama/Polkadot Validator (Pruned Node)

The main setup playbook is:

```bash
ansible-playbook -i inventory polkadot.yml -e "target=VALIDATOR_TARGET"
```

Notice that you need to specify a target when you run this playbook (and other playbooks in this repo, as described in the next section). `VALIDATOR_TARGET` is a placeholder that could be a host (`kusama1`, `kusama2`, `polkadot1`, `polkadot2`, etc), or a group (`kusama`, `polkadot`). This is intentionally designed to:

1. Prevent you from updating all nodes by mistake
2. Allow you to experiment a move on a low-risk node before rolling out to the whole cluster

## Other Playbooks for Different Purposes

The most commonly used playbooks are:

| Playbook           | Description                                                               |
| ------------------ | ------------------------------------------------------------------------- |
| `prepare.yml `     | Do the prep work, such as ufw, node_exporter and promtail                 |
| `polkadot.yml`     | Install Kusama/Polkadot node                                              |
| `key_rotation.yml` | Rotate session keys the easy way without you ssh into the server yourself |

That's it, folks!
