# Polkdadot Validation Node Ansible Setup

This repo is to set up Polkadot Validation node. This repo is a heavily modified version of https://github.com/w3f/polkadot-validator-setup

## Set Up Kusama/Polkadot Validator

Make sure that you have a production inventory file. You can start by copying the sample inventory file (included in the repo) if you start from scratch. The sample file gives you a good idea about how to define the inventory.

```bash
cp inventory.sample inventory
```

Make sure that you are familiar with the files in group_vars. They often need to change to stay up to date with the latest release. It is very possible that you will get an error on checksum of data restore in your first attempt, as the data screenshot by polkashots.io is updated frequently.

The key validator ansible file is polkadot_full_setup.yml, which will set up a new fresh validator from scratch.

```bash
ansible-playbook -i inventory polkadot_full_setup.yml -e "target=VALIDATOR_TARGET"
```

VALIDATOR_TARGET could be a host (kusama1, kusama2, polkadot1, polkadot2, etc), could be a group (kusama, polkadot), or all validators (validators)

The most commonly used playbooks are:

| Playbook                | Description                                                                              |
| ----------------------- | ---------------------------------------------------------------------------------------- |
| polkadot_full_setup.yml | Run the initial full setup                                                               |
| polkadot_prepare.yml    | Do the prep work, such as firewall, set up proxy, copy service files, create users, etc. |
| polkadot_update.yml     | Update the polkadot binary and restart service. You probably need to use it regularly    |
| polkadot_restore.yml    | Restore the polkadot database with screenshot. Only useful for initial setup             |
| node_exporter.yml       | Update Node Exporter                                                                     |
| process_exporter.yml    | Update Process Exporter                                                                  |

The less commonly used playbooks are:

| Playbook                     | Description                                                                  |
| ---------------------------- | ---------------------------------------------------------------------------- |
| polkadot_backup_keystore.yml | Backup keystore (Not sure about use case)                                    |
| polkadot_clean_logs.yml      | Clean journal logs (Probably useful when disk is full)                       |
| polkadot_restart.yml         | Restart polkadot ad hoc (Probably useful when server runs hot for no reason) |
| polkadot_stop.yml            | Stop polkadot ad hoc                                                         |
| polkadot_rorate_keys.yml     | Rotate session keys the easy way                                             |

## Update All Servers

Often, you might want to update all servers. That's easy. Just run:

```bash
ansible-playbook -i inventory all_apt_update.yml
```

### table to details all the other features.

## TODO

A script to check the latest release.

make sure to use the extra varraible to set target

## TODO

Make a shell script that accept a target varaible and it will set everything up as a validators
