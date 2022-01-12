# Polkadot Validation Node Ansible Setup

This repo is to set up the Polkadot Validation node. This repo is heavily influenced by https://github.com/w3f/polkadot-validator-setup, the official Polkadot secure setup guide.

## Motivation

While the official setup is very comprehensive, it can be overwhelming for "small" validators (myself included) who do not care much about using Terraform on the infrastructure layer. I took the Ansible part of the script and updated it:

1. The setup is more opinionated, thus the script is simpler by avoiding many "if" statements. It is tailored for Ubuntu only, but you should be able to get it working on other Linux distributions with some revisions.
2. It is more opinionated about node monitoring by recommending Node Exporter, Processor Exporter, and Promtail (for centralized log monitoring). I also have a companion Ansible script (https://github.com/polkachu/server-monitoring) that installs Prometheus, Grafana, and Loki to set up such a centralized monitoring server. This setup will make your life easier if you eventually move from a "small" validator to running a cluster of Polkadot/Kusama nodes.
3. The setup assumes that you will start from an archived node snapshot provided by https://polkashots.io. It is much simpler and less error-prone than Rust compiling. Highly recommended. In fact, we at Polkachu are currently planning to offer such archived node snapshots to provide redundancy to the community.
4. Since it has happened twice already, I have included a configuration to help you roll back to version `0.8.30` in the `group_vars/polkadot.yml` file.

## Summary

You run one playbook and set up a Kusama/Polkadot node. Boom!

```bash
ansible-playbook -i inventory polkadot_full_setup.yml -e "target=VALIDATOR_TARGET"
```

But before you rush with this easy setup, you probably want to read on so you understand the structure of this Ansible program and all the features it offers.

## Some Preparations

First of all, some preparation is in order.

Make sure that you have a production inventory file with your confidential server info. You will start by copying the sample inventory file (included in the repo). The sample file gives you a good idea on how to define the inventory.

```bash
cp inventory.sample inventory
```

Needless to say, you need to update the dummy values in the inventory file. For each Kusama/Polkadot node, you need to update:

1. Server IP: Your server public IP
2. validator_name: This is the node name that will show up on telemetry monitoring board. It is especially important if you want to participate in the Thousand Validators Program. For us, we use something like `polkachu-kusama-01` and `polkachu-polkadot-02` to keep it unique and organized.
3. log_name: This is for your internal central monitoring server. We just use something like `kusama1` and `polkadot2` to keep it simple.
4. telemetryUrl: Most likely you will use `wss://telemetry-backend.w3f.community/submit/`
5. archive_node (optional): Set this to true if you want to run an archive node. An archive node is not required for a validator. An archive node has the complete chain data and requires much larger storage space. Most validators do not need an archive node.
6. chain_path (optional): You can set an alternative path to store chain data. This is especially useful when you run an archive node and want to store chain data on a mounted disk. A mounted disk offers more flexibility when you want to wrap disk, increase or decrease disk size, etc.

You will also need to update:

1. ansible_user: The sample file assumes `ansible`, but you might have another username. Make sure that the user has `sudo` privilege.
2. ansible_port: The sample file assumes `22`. But if you are like me, you will have a different ssh port other than `22` to avoid port sniffing.
3. ansible_ssh_private_key_file: The sample file assumes `~/.ssh/id_rsa`, but you might have a different key location.
4. log_monitor: Enter your monitor server IP. It is most likely a private IP address if you use a firewall around your private virtual cloud (VPC).

It is beyond the scope of this guide to help you create a sudo user, alternate ssh port, create a private key, install Ansible on your machine, etc. You can do a quick online search and find the answers. In my experience, Digital Ocean have some quality guides on these topics. Stack Overflow can help you trouble-shoot if you are stuck.

## Basic Cluster Structure

The basic cluster structure is:

1. Name each Kusama node as `kusama1`, `kusama2`, etc. Group all Kusama nodes into `kusama` group.
2. Name each Polkadot node as `polkadot1`, `polkadot2`, etc. Group all Polkadot nodes into `polkadot` group.
3. Group all nodes into a `validators` group.

The structure allows you to target `vars` to each node, or either Kusama or Polkadot cluster, or the whole cluster.

Make sure that you are familiar with the files in the `group_vars` folder. They follow this clustered structure closely. The files in this folder often need to be changed to stay up to date with the latest releases. I, for one, bump these program versions religiously so I live on the cutting edge!

## Main Playbook to Set Up a Kusama/Polkadot Validator (Pruned Node)

The key Ansible playbook is `polkadot_full_setup.yml`. It will set up a fresh validator from scratch. Notice that it will restore from a snapshot from https://polkashots.io. It is very possible that you will get an error on the checksum of data to restore in your first attempt because the snapshot is updated regularly. When this happens, update the files accordingly.

The main setup playbook is:

```bash
ansible-playbook -i inventory polkadot_full_setup.yml -e "target=VALIDATOR_TARGET"
```

Notice that you need to specify a target when you run this playbook (and other playbooks in this repo, as described in the next section). `VALIDATOR_TARGET` is a placeholder that could be a host (`kusama1`, `kusama2`, `polkadot1`, `polkadot2`, etc), a group (`kusama`, `polkadot`), or all validators (`validators`). This is intentionally designed to:

1. Prevent you from updating all nodes by mistake
2. Allow you to experiment a move on a low-risk node before rolling out to the whole cluster

## Main Playbook to Set Up a Kusama/Polkadot Archive Node

The main setup playbook is:

```bash
ansible-playbook -i inventory polkadot_full_archive_node_setup.yml -e "target=VALIDATOR_TARGET"
```

Most validators DO NOT need archive node.

## A Pitfall

We introduced pruned node / archive node toggle in the version 0.2.0 release. The database for pruned node and archive node is not compatible. If you have trouble start your `polkadot` service, a simple trouble-shooting method is just to delete the whole polkadot `db` directory.

## Other Playbooks for Different Purposes

The most commonly used playbooks are:

| Playbook                  | Description                                                                                |
| ------------------------- | ------------------------------------------------------------------------------------------ |
| `polkadot_full_setup.yml` | Run the initial full setup                                                                 |
| `polkadot_prepare.yml `   | Do the prep work, such as firewall, set up a proxy, copy service files, create users, etc. |
| `polkadot_update.yml`     | Update the Polkadot binary and restart the service. You probably need to use it regularly  |
| `polkadot_restore.yml`    | Restore the Polkadot database with a screenshot. Only useful for initial setup             |
| `node_exporter.yml`       | Update Node Exporter                                                                       |
| `process_exporter.yml`    | Update Process Exporter                                                                    |
| `promtail.yml`            | Update Promtail                                                                            |

The less commonly used playbooks are:

| Playbook                       | Description                                                                           |
| ------------------------------ | ------------------------------------------------------------------------------------- |
| `polkadot_backup_keystore.yml` | Backup Keystore (Not sure about use case)                                             |
| `polkadot_clean_logs.yml`      | Clean journal logs (Probably useful when the disk is full)                            |
| `polkadot_restart.yml`         | Restart Polkadot ad hoc (Probably useful when server runs wild for no obvious reason) |
| `polkadot_stop.yml`            | Stop Polkadot ad hoc                                                                  |
| `polkadot_rotate_key.yml`      | Rotate session keys the easy way without you ssh into the server yourself             |
| `snapshot_script.yml`          | If you intend to use the node to take snapshot, then this script is for you           |

## Update All Servers

One more thing! Sometimes you want to install all apt patches on all machines. I provide you with a simple playbook. Just run:

```bash
ansible-playbook -i inventory all_apt_update.yml
```

That's it, folks!

## Tips/Nominations Accepted

- DOT: `15ym3MDSG4WPABNoEtx2rAzBB1EYWJDWbWYpNg1BwuWRAQcY`
- KSM: `CsKvJ4fdesaRALc5swo5iknFDpop7YUwKPJHdmUvBsUcMGb`
