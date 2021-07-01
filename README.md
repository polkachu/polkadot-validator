# Polkdadot Validation Node Ansible Setup

This repo is to set up Polkadot Validation node. This repo is a heavily influenced by https://github.com/w3f/polkadot-validator-setup, the official Polkadot secure setup guide.

## Motivation

While the official setup is very comprehensive, it can be overwhelming for "small" validators (myself included) who do not care much about using Terraform on the infrastructure layer. I took the Ansible part of the script and updated it:

1. The setup is more opinionated, thus the script is simplier by avoiding many "if" statements. It is tailored for Ubuntu only, but you should be able to get it working on other Linux distribution with some revisions.
2. It is more opinionated about node monitoring by recommending Node Exporter, Processor Exporter, and Promtail (for centralized log monitoring). I also have a companion Ansible script (https://github.com/polkachu/server-monitoring) that installs Prometheus, Grafana and Loki to set up such a centralized monitoring server. This setup will make your life easier if you eventually move from a "small" validator to running a cluster of Polkadot/Kusama nodes.
3. The setup assumes that you will start from an archived node snapshot provided by https://polkashots.io. It is much simpler and less error-prone than Rust compiling. Highly recommended. In fact, we at Polkachu is currently planning to offer such archived node snapshots to provide redundency to the community.
4. Since it has happened twice already, I have included a configuration to help you roll back to version `0.8.30` in the `group_vars/polkadot.yml` file.

## Set Up Kusama/Polkadot Validator

Make sure that you have a production inventory file. You will start by copying the sample inventory file (included in the repo). The sample file gives you a good idea on how to define the inventory.

```bash
cp inventory.sample inventory
```

The basic cluster structure is:

1. Name each Kusama node as `kusama1`, `kusama2`, etc. Group all Kusama nodes into `kusama` group.
2. Name each Polkadot node as `polkadot1`, `polkadot2`, etc. Group all Polkadot nodes into `polkadot` group.
3. Group all nodes into `validators` group.

The structure allows you to target `vars` to each node, either Kusama or Polkadot cluster, or all nodes.

Make sure that you are familiar with the files in the `group_vars` folder. They follow this cluster structure closely. The files in this folder often need to change to stay up to date with the latest release.

The key validator ansible file is `polkadot_full_setup.yml`, which will set up a fresh validator from scratch. Notice that the full setup will restore from a snapshot from https://polkashots.io. It is very possible that you will get an error on checksum of data restore in your first attempt, because the snapshot is updated regularly. When this happens, update the files accordingly.

The main setup playbook is:

```bash
ansible-playbook -i inventory polkadot_full_setup.yml -e "target=VALIDATOR_TARGET"
```

Notice that you need to specify a target when you run this playbook (and other playbooks in this repo, as described in the next section). `VALIDATOR_TARGET` is a placeholder that could be a host (`kusama1`, `kusama2`, `polkadot1`, `polkadot2`, etc), a group (`kusama`, `polkadot`), or all validators (`validators`). This is intentionally designed to:

1. Prevent you from updating all nodes by mistake
2. Allow you to expirement a move on a low-risk node before rolling out to the whole cluster

## Various Playbooks for Different Purposes

The most commonly used playbooks are:

| Playbook                  | Description                                                                              |
| ------------------------- | ---------------------------------------------------------------------------------------- |
| `polkadot_full_setup.yml` | Run the initial full setup                                                               |
| `polkadot_prepare.yml `   | Do the prep work, such as firewall, set up proxy, copy service files, create users, etc. |
| `polkadot_update.yml`     | Update the polkadot binary and restart service. You probably need to use it regularly    |
| `polkadot_restore.yml`    | Restore the polkadot database with screenshot. Only useful for initial setup             |
| `node_exporter.yml`       | Update Node Exporter                                                                     |
| `process_exporter.yml`    | Update Process Exporter                                                                  |
| `promtail.yml`            | Update Promtail                                                                          |

The less commonly used playbooks are:

| Playbook                       | Description                                                                           |
| ------------------------------ | ------------------------------------------------------------------------------------- |
| `polkadot_backup_keystore.yml` | Backup keystore (Not sure about use case)                                             |
| `polkadot_clean_logs.yml`      | Clean journal logs (Probably useful when disk is full)                                |
| `polkadot_restart.yml`         | Restart polkadot ad hoc (Probably useful when server runs wild for no obvious reason) |
| `polkadot_stop.yml`            | Stop polkadot ad hoc                                                                  |
| `polkadot_rorate_keys.yml`     | Rotate session keys the easy way without you ssh into the server yourself             |

## Update All Servers

One more thing! Sometimes you want to install all apt patches on all machines. I provides you with a simple playbook. Just run:

```bash
ansible-playbook -i inventory all_apt_update.yml
```

That's it, folks!

## Tips/Nominations Accepted

- DOT: `15ym3MDSG4WPABNoEtx2rAzBB1EYWJDWbWYpNg1BwuWRAQcY`
- KSM: `CsKvJ4fdesaRALc5swo5iknFDpop7YUwKPJHdmUvBsUcMGb`
