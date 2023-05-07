# PulseChain Testnet Validator Node Setup Scripts

![pls-testnet-validator-htop](https://user-images.githubusercontent.com/100790377/229965674-75593b5a-3fa6-44fe-8f47-fc25e9d3ce21.png)

This will help you setup [PulseChain](www.pulsechain.com) Testnet v4 and plans are to update it to support [PulseChain](www.pulsechain.com) Mainnet as well after it launches. Since it is a fork of ETH 2.0, all the same methods and scripts can be modified to work fine and automate Ethereum node setup as well.

**Please read ALL the instructions as they will explain and tell you how to run these scripts and the caveats.**

To download these scripts on your server, you can `git clone https://github.com/rhmaxdotorg/pulsechain-validator.git`.

After you download the code, you may need to `chmod +x *.sh` to make all the scripts executable and able to run on the system.

# Description

Scripts and guidance available include...
- PulseChain Validator setup (~85% entire process automated)
- Use the staking deposit client
- Grafana and Prometheus monitoring setup
- Setting up an AWS cloud server
- Updating your client versions to the latest
- Updating your fee recipient and IP address
- Enabling local RPC for Metamask

The setup script installs pre-reqs, golang, rust, go-pulse (geth fork) and lighthouse on a fresh, clean Ubuntu OS for getting a PulseChain Testnet (V4) Validator Node setup and running with **Geth (go-pulse)** and **Lighthouse** clients.

Clients are running on the same machine and not in docker containers (such as the method other scripts use). There are advantages and disadvantages to containerizing the clients vs running them on the host OS. You can also automate deployments with Terraform on AWS, however the scripts are meant to make it pretty easy to spin up and tear down new validator machines once you have the hardware up with access to a fresh install of Ubuntu Linux.

There are other helper scripts that do various things, check the notes for each one specifically for more info.

You can run **pulsechain-validator-setup.sh** to setup your validator clients and **monitoring-setup.sh** afterwards to install the graphs and monitoring software.

Note: the pulsechain validator setup script doesn't install monitoring/metrics packages, however a script to do that is provided. It would need to run the validator setup script AND THEN run the monitoring-setup.sh script provided. Do not run the monitoring script before installing your validator clients. See details in the [Grafana or Prometheus](https://github.com/rhmaxdotorg/pulsechain-validator#setting-up-monitoring-with-prometheus-and-grafana) section.

Table of Contents
=================

* [PulseChain Testnet Validator Node Setup Scripts](#pulsechain-testnet-validator-node-setup-scripts)
* [Description](#description)
* [Table of Contents](#table-of-contents)
* [Walkthrough](#walkthrough)
* [Usage](#usage)
   * [Command line options](#command-line-options)
* [Environment](#environment)
* [Hardware](#hardware)
* [After running the script](#after-running-the-script)
* [Debugging](#debugging)
   * [Check the Blockchain Sync Progress](#check-the-blockchain-sync-progress)
      * [Geth](#geth)
      * [Lighthouse](#lighthouse)
   * [Look at Client Service Status](#look-at-client-service-status)
* [Reset Validator Script](#reset-validator-script)
* [New Server Helper Script](#new-server-helper-script)
* [Client Update Script](#client-update-script)
* [Fee Recipient and IP Address Update Script](#fee-recipient-and-ip-address-update-script)
* [RPC Interface Script](#rpc-interface-script)
* [Snapshot Helper Script](#snapshot-helper-script)
* [AWS Cloud Setup](#aws-cloud-setup)
* [Staking Deposit Client Walkthrough](#staking-deposit-client-walkthrough)
* [Details for all PulseChain clients (/w Ethereum Testnet notes)](#details-for-all-pulsechain-clients-w-ethereum-testnet-notes)
* [Setting up monitoring with Prometheus and Grafana](#setting-up-monitoring-with-prometheus-and-grafana)
   * [Web UI setup](#web-ui-setup)
* [Community Guides, Scripts and Dashboards](#community-guides-scripts-and-dashboards)
* [Security](#security)
* [Networking](#networking)
   * [Server](#server)
   * [Home Router](#home-router)
   * [AWS Cloud](#aws-cloud)
* [Graffiti](#graffiti)
* [Withdrawals](#withdrawals)
   * [Overview](#overview)
   * [Withdrawal Keys](#withdrawal-keys)
   * [Exiting](#exiting)
* [Backups](#backups)
   * [Home](#home)
   * [Cloud](#cloud)
* [FAQ](#faq)
* [Additional Resources and References](#additional-resources-and-references)

# Walkthrough
Check out these videos for further explanations and code walkthroughs.
- https://www.youtube.com/watch?v=6-ePJXAUfdg
- https://www.youtube.com/watch?v=X0TnkLt4E3w
- https://www.youtube.com/watch?v=QqcDs8llyyw
- https://www.youtube.com/watch?v=YFOxf4B27Zs
- https://www.youtube.com/watch?v=9Yibmetppcs
- https://www.youtube.com/results?search_query=rhmax+validator+ama

# Usage

```
$ ./pulsechain-validator-setup.sh [0x...YOUR NETWORK FEE ADDRESS] [12.89...YOUR SERVER IP ADDRESS]
```

## Command line options

- **NETWORK FEE ADDRESS** is the FEE_RECIPIENT value for --suggested-fee-recipient to a wallet address you want to recieve priority fees from users whenever your validator proposes a new block (else it goes to the burn address)

- **SERVER_IP_ADDRESS** to your validator server's IP address

Note: you may get prompted throughout the process to hit [Enter] for OK and continue the process

For example when running Ubuntu on AWS EC2 cloud service, you can expect to hit OK on kernel upgrade notice, [Enter] or "1" to continue Rust install process and so on.

**If you encounter errors running the script and want to run the script again, use the [Reset the Validator](https://github.com/rhmaxdotorg/pulsechain-validator/blob/main/README.md#reset-validator-script) BEFORE running it over and over again.**

Just make sure you know what you're doing and manually edit the reset script to bypass the "I don't know what I'm doing" check. It's very straightforward, just read the code, acknowledge you know what the script it doing and **change I_KNOW_WHAT_I_AM_DOING=false to true to get it to run**.

# Environment
Tested on **Ubuntu 22.04** (Amazon AWS EC2 /w M2.2Xlarge cloud server) running as a non-root user (ubuntu) with sudo privileges.

# Hardware
The consensus on the **minimum recommended requirements** to run a validator seem to be **32gb RAM, 2TB disk and plenty of processing power (quadcore, xeon/ryzen, 4-8 vCPUs and such)**. These can come in the form of buying or building your own server and paying an upfront cost, utilities and maintenance OR renting a server from a VPS/cloud provider such as **Amazon AWS (M2.2Xlarge server)** and paying monthly to use their platform and resources. Both have advantages and disadvantages as well as varying time, monetary and management costs.

Could you get by with an old PC under your desk with a $50 battery backup? Maybe, but that would not be *recommended*. I'd rather not skimp on hardware for things that I would plan to run for years and pay for the peace of mind of not worrying about what I'm going to do if X fails one day, wishing I'd started with stronger foundations. If you try and do it right the first time, you might save a lot of time and headache.

# After running the script

The script automates a roughly estimated ~85% of what it takes to get the validator configured, but there's still a few manual steps you need to do to complete the setup and get the validator on the network.

**Generate validator keys with deposit tool and import them into Lighthouse**

**Run the staking deposit client (ON A DIFFERENT MACHINE, see notes below for details)**
```
$ sudo apt install -y python3-pip
$ git clone https://gitlab.com/pulsechaincom/staking-deposit-cli.git
$ cd staking-deposit-cli && pip3 install -r requirements.txt && sudo python3 setup.py install
$ ./deposit.sh new-mnemonic --chain=pulsechain-testnet-v4 --eth1_withdrawal_address=0x... (ENTER THE CORRECT WALLET ADDRESS TO WITHDRAWAL YOUR FUNDS)
```

Note: it is **VERY IMPORTANT** that you use a withdrawal wallet address that you have access to and is SECURE for a long time. Otherwise you may lose all your deposit funds.

**Then follow the instructions from there, copy them over to the validator and import into lighthouse AS THE NODE USER (not the 'ubuntu' user on ec2).**
```
$ sudo cp -R validator_keys /home/node
$ sudo chown -R node:node /home/node/validator_keys
$ sudo -u node bash

(as node user)
$ /opt/lighthouse/lighthouse/lh account validator import --directory ~/validator_keys --network=pulsechain_testnet_v4

enter password to import validator(s)

(exit and back as ubuntu user)
```

Note: generate your keys on a different, secure machine (NOT on the validator server) and transfer them over for import. **See the Security section for more references on why this is important.** AWS even offers a [free tier](https://aws.amazon.com/free/free-tier-faqs/) option that allows you to spin up and use VMs basically for free for a certain period of time, so you could use that for quick and easy tiny VMs running Ubuntu Linux (not beefy enough to be a validator, but fine for small tasks and learning).

You can use the `scp` command to copy validator keys over the network (encrypted), USB stick (if hardware is local, not vps/cloud) OR use this base64 encoding trick for a copy and paste style solution such as the following. Note: this is advanced and you need to pay attention to be successful with it. If you're not confident you can do it, **better to use scp or USB methods**.

**On disposable VM, live CD or otherwise emphemeral filesystem**

```
sudo apt install -y unzip zip
zip -r validator_keys.zip validator_keys
base64 -w0 validator_keys.zip > validator_keys.b64
cat validator_keys.b64 (and copy the output)
```

**On your validator server**
```
cat > validator_keys.b64 <<EOF
Paste the output
[Enter] + type “EOF” + [Enter]
base64 -d validator_keys.b64 > validator_keys.zip
unzip validator_keys.zip
```

Also see [Validator Key Generation and Management](https://docs.google.com/document/d/1tl_nql6-Bqyo5yqFDJ2aqjAQoBAK0FtcCYSKpGXg0hw/edit) for more guidance.

**Start the beacon and validator clients**
```
$ sudo systemctl daemon-reload
$ sudo systemctl enable lighthouse-beacon lighthouse-validator
$ sudo systemctl start lighthouse-beacon lighthouse-validator
```

If you want to look at lighthouse debug logs (similar to geth)

```
$ journalctl -u lighthouse-beacon.service (with -f to get the latest logs OR without it to get the beginning logs)
$ journalctl -u lighthouse-validator.service
```

**Once the blockchain clients are synced, you can make your 32m tPLS deposit (per validator)**

You can have multiple on one machine. The deposit is made @ https://launchpad.v4.testnet.pulsechain.com to get your validator activated and participating on the network.

If you do the deposit before the clients are fully synced and ready to go, then you risk penalities as your validator would join the network, but due to not being synced, unable to participate in validator duties (until it's fully synced).

Now let's get validating! @rhmaximalist

# Debugging

## Check the Blockchain Sync Progress

### Geth
```
$ curl -s http://localhost:8545 -H "Content-Type: application/json" --data '{"jsonrpc":"2.0","method":"eth_syncing","params":[],"id":67}' | jq

{
  "jsonrpc": "2.0",
  "id": 67,
  "result": {
  "currentBlock": "0xffe4e3", // THIS IS WHERE YOU ARE
  "highestBlock": "0xffe8fa", // THIS IS WHERE YOU’RE GOING
  [full output was truncated for brevity]
  }
}
```

So you can compare the current with the highest to see how far you are from being fully sync’d. Or is result=false, you are sync'd.

```
$ curl -s http://localhost:8545 -H "Content-Type: application/json" --data "{\"jsonrpc\":\"2.0\",\"method\":\"eth_syncing\",\"params\":[],\"id\":67}" | jq
{
  "jsonrpc": "2.0",
  "id": 67,
  "result": false
}
```
### Lighthouse
```
$ curl -s http://localhost:5052/lighthouse/ui/health | jq
{
  "data": {
	"total_memory": XXXX,
	"free_memory": XXXX,
	"used_memory": XXXX,
	"os_version": "Linux XXXX",
	"host_name": "XXXX",
	"network_name": "XXXX",
	"network_bytes_total_received": XXXX,
	"network_bytes_total_transmit": XXXX,
	"nat_open": true,
	"connected_peers": 0, // PROBLEM
	"sync_state": "Synced"
  [full output was truncated for brevity]
  }
}
```


```
$ curl -s http://localhost:5052/lighthouse/syncing | jq
{
  "data": "Synced"
}
```
## Look at Client Service Status

```
$ sudo systemctl status geth lighthouse-beacon lighthouse-validator

● geth.service - Geth (Go-Pulse)
     Loaded: loaded (/etc/systemd/system/geth.service; enabled; vendor preset: enabled)
     [some output truncated for brevity]

Apr 00 19:30:20 server geth[126828]: INFO Unindexed transactions blocks=1 txs=56   tail=14,439,524 elapsed=2.966ms
Apr 00 19:30:30 server geth[126828]: INFO Imported new potential chain segment blocks=1 txs=35   mgas=1.577  elapsed=21.435ms     mgasps=73.569  number=16,789,524 hash=xxxxd7..xxx>
Apr 00 19:30:30 server geth[126828]: INFO Chain head was updated                   number=16,789,xxx hash=xxxxd7..cdxxxx root=xxxx9c..03xxxx elapsed=1.345514ms
Apr 00 19:30:30 server geth[126828]: INFO Unindexed transactions blocks=1 txs=96   tail=14,439,xxx elapsed=4.618ms

● lighthouse-beacon.service - Lighthouse Beacon
     Loaded: loaded (/etc/systemd/system/lighthouse-beacon.service; enabled; vendor preset: enabled)
     [some output truncated for brevity]

Apr 00 19:30:05 server lighthouse[126782]: INFO Synced slot: 300xxx, block: 0x8355…xxxx, epoch: 93xx, finalized_epoch: 93xx, finalized_root: 0x667f…707b, exec_>

Apr 00 19:30:10 server lighthouse[126782]: INFO New block received root: 0xxxxxxxxxf5e1364e34de345ab72bf1632e814915eb3fdc888e5b83aaxxxxxxxx, slot: 300061

Apr 00 19:30:15 server lighthouse[126782]: INFO Synced slot: 300xxx, block: 0x681e…xxxx, epoch: 93xx, finalized_epoch: 93xx, finalized_root: 0x667f…707b, exec_>

● lighthouse-validator.service - Lighthouse Validator
     Loaded: loaded (/etc/systemd/system/lighthouse-validator.service; enabled; vendor preset: enabled)
     [some output truncated for brevity]

Apr 00 19:30:05 server lighthouse[126779]: Apr 06 19:30:05.000 INFO Connected to beacon node(s)             synced: X, available: X, total: X, service: notifier
Apr 00 19:30:05 server lighthouse[126779]: INFO All validators active slot: 300xxx, epoch: 93xx, total_validators: X, active_validators: X, current_epoch_proposers: 0, servic>
```

## Look at client debug logs

For example, `journalctl` will let you look at each client's debug logs. Recommend it with the `-f` option to get the latest logs.

```
$ journalctl -u geth.service
$ journalctl -u lighthouse-beacon.service
$ journalctl -u lighthouse-validator.service -f (with -f to get the latest logs OR without it get the beginning logs)
```

# Reset Validator Script
This helper script deletes all your validator data so you can try the setup again if you want a fresh install or feel like you made an error.

Be careful! It deletes and resets things, so read the code and make sure you understand what it does before using it.

# New Server Helper Script
Just some nice-to-haves if you're using the AWS Cloud for your validator server.

# Client Update Script

It stops the client services, pulls updates from Gitlab, rebuilds the clients and starts the services back again. Only supports Geth and Lighthouse.

Note: **validator will be offline for likely 1 hour while the updates are taking place**, so before you run this script, make sure you understand and are OK with that.

# Fee Recipient and IP Address Update Script

This one allows you to update the network fee recipient and server IP address for Lighthouse. These were specified during the initial PulseChain validator setup, however both of them may change for you over time, so the script allows you to easily update them and restart the clients.

# RPC Interface Script

Enables the RPC interface so you can use your own node in Metamask (support for Firefox only). Running your own node and using it can be used for testing purposes, not relying on public servers or bypassing slow, rate limited services by "doing it yourself".

**Do not expose your RPC publicly unless you know what you're doing.** This script helps you more securely expose it to your own local environment for your own use.

If your RPC is...
- On the same machine as Metamask, you can point it at 127.0.0.1
- On VPS/cloud server, you can use SSH port forwarding and then point it at 127.0.0.1
- On a different machine on your local network, open the port on the local firewall and point it at that local IP address

**Add your server to Metamask**

Click the Network drop-down, then Add Network and Add a Network Manaully.

- Network name: Local PLS
- New RPC URL: http://local-network-server-IP:8564 OR http://127.0.0.1:8546 (running same machine OR port forwarded)
- Chain ID: 943 (for testnet v4)
- Currency symbol: tPLS
- Block explorer URL: https://scan.v4.testnet.pulsechain.com
- Save

Now you can use your own node for transactions on the network that your validator is participating in.

# Snapshot Helper Script

Takes a snapshot of blockchain data on a fully synced validator so it can be copied over and used to bootstrap a new validator. Clients must be stopped until the snapshot completes, afterwards they will be restarted so the validator can resume normal operation.

After running the script, copy the geth.tar.xz and lighthouse.tar.xz (compressed blockchain data, kinda like ZIP files) over to the new validator server (see scp demo below OR use a USB stick).

```
$ scp -i key.pem geth.tar.xz ubuntu@new-validator-server-ip:/home/ubuntu
$ scp -i key.pem lighthouse.tar.xz ubuntu@new-validator-server-ip:/home/ubuntu
```

Copying over the network could take anywhere from 1 hour to a few hours (depending on the bandwidth of your server's network).

Then you can run the following commands ON THE NEW SERVER

```
$ tar -xJf geth.tar.xz
$ tar -xJf lighthouse.tar.xz
$ sudo chown -R node:node data beacon
$ sudo mv data /opt/geth
$ sudo mv beacon /opt/lighthouse/data
```

The geth.tar.xz file is likely going to be > 100gb and the lighthouse compressed file probably smaller, but prepare for ~200gb of data total for the transfer.

Note: this should work fine for Ethereum too as it's just copying the blockchain data directories for Geth and Lighthouse, but the scenario is technically untested. Also, this relies on the new validator setup (which you are copying the snapshot to) to be setup with this repo's setup script.

# AWS Cloud Setup
* [How to run a cloud server on AWS](https://docs.google.com/document/d/1eW0SDT8IvZrla7gywK32Rl3QaQtVoiOu5OaVhUKIDg8/edit)

AWS also offers a [free tier](https://aws.amazon.com/free/free-tier-faqs/) option that allows you to spin up Linux VMs for free for a certain period of time, so you could use that for quick and easy tiny VMs running Ubuntu Linux. They are not beefy enough to be a validator, so that's not an option, but they are fine for small tasks and learning. You just need to sign up for an account and follow the instructions in the above document, except choose Free Tier options instead of the validator hardware configuration as described.

# Staking Deposit Client Walkthrough

* [Validator Key Generation and Management](https://docs.google.com/document/d/1tl_nql6-Bqyo5yqFDJ2aqjAQoBAK0FtcCYSKpGXg0hw/edit)

# Details for all PulseChain clients (/w Ethereum Testnet notes)
* [Geth, Erigon, Prysm and Lighthouse](https://docs.google.com/document/d/1RkAWt0Q_DmYpnykHFM4Qf5ItDLPLi-kaj1PDG74Mftg/edit)

# Setting up monitoring with Prometheus and Grafana

The **monitoring-setup.sh** and **reset-monitoring.sh** automate most of the setup for grafana and prometheus as well as let you reset (or remove) the monitoring, respectively.

**You need to run the validator setup script FIRST and then use the monitoring setup script to "upgrade" the install with monitoring.**

## Web UI setup

After running the monitoring setup script, you must finish the configuration at the Grafana portal and import the dashboards.

The standard config assumes you are not sharing the validator server with other people (local user accounts). Otherwise, it’s recommended for security reasons to set up further authentication on the monitoring services. TL;DR you should be the only one with remote access to your validator server, so ensure your keys and passwords are safe and do not share them with anyone for any reason.

You can setup grafana for secure access externally as opposed to the less secure way of forwarding port 3000 on the firewall and open it up to the world, which could put your server at risk next time Grafana has a security bug that anyone interested enough can exploit.

```
ssh -i key.pem -N ubuntu@validator-server-IP -L 8080:localhost:3000
```

Then open a browser window on your computer and login to grafana yourself without exposing it externally to the world. Magic, huh!

Go to http://localhost:8080 and login with admin/admin (as the initial username/password). It will then ask you to set a new password, make it a good one.

In the lower left bar area, click the gear box -> Data Sources -> Add Data Source.
- Select Prometheus
- URL: http://localhost:9090
- Click Save & Test
- It should say “Datasource is working” in green

Use your mouse cursor to hover over the Dashboards icon (upper left bar area, 4 squares icon).
- Select Import
- Upload each JSON dashboard

Geth
- Download it @ https://gist.githubusercontent.com/karalabe/e7ca79abdec54755ceae09c08bd090cd/raw/3a400ab90f9402f2233280afd086cb9d6aac2111/dashboard.json to import
- Name: Geth
- Datasource: Prometheus (default)
- Click Import
- Click Save button (and confirm Save) in upper right toolbar
- Repeat for next dashboard

Lighthouse VC
- Download it @ https://raw.githubusercontent.com/sigp/lighthouse-metrics/master/dashboards/ValidatorClient.json to import
- Name: Lighthouse VC
- Datasource: Prometheus (default)
(same steps as previous)

Lighthouse Beacon
- Download it @ https://raw.githubusercontent.com/sigp/lighthouse-metrics/master/dashboards/Summary.json to import
- Name: Lighthouse Beacon
- Datasource: Prometheus (default)
(same steps as previous)

There's also this ETH forked Dashboard for PulseChain @ https://github.com/raskitoma/pulse-staking-dashboard which has really good stats!

Staking Dashboard
- Download it @ https://raw.githubusercontent.com/raskitoma/pulse-staking-dashboard/main/Yoldark_ETH_staking_dashboard.json to import
- Name: Staking Dashboard
- Datasource: Prometheus (default)
(same steps as previous)

Now you can browse the Dashboards and see various stats and data!

Also see the guides below for additional help (scripts were mostly based on those instructions)
- https://www.coincashew.com/coins/overview-eth/guide-or-how-to-setup-a-validator-on-eth2-mainnet/part-i-installation/monitoring-your-validator-with-grafana-and-prometheus
- https://schh.medium.com/port-forwarding-via-ssh-ba8df700f34d
- https://github.com/raskitoma/pulse-staking-dashboard

You can also setup **email alerts** on Grafana. See guide at the link below.
- https://thriftly.io/docs/components/Thriftly-Deployment-Beyond-the-Basics/Metrics-Prometheus/Creating-Receiving-Email-Alerts.html

# Community Guides, Scripts and Dashboards
- https://www.gammadevops.com/p/validator-setup
- https://gitlab.com/davidfeder/validatorscript/-/blob/64f37685908a78c5337f8d3dc951f7f01f251697/PulseChain_V4_Script.txt
- https://www.hexpulse.info/docs/node-setup.html
- https://togosh.medium.com/pulsechain-validator-setup-guide-70edae00b344
- https://github.com/tdslaine/install_pulse_node
- https://github.com/raskitoma/pulse-staking-dashboard

# Security

If running a validator in the cloud, there's already isolation away from your home network and the devices connected to it. However, if running a validator on your home network, the game is to keep attackers off of your home network. This is much easier when you're not inviting the public to connect to your server that sits on your network at home, but with validators, you're naturally exposing infrastructure running on your own network, which may be the same one you connect your personal devices to as well.

Recommended that if running a validator at home, you isolate it from everything else on your home network using another router into the mix, cascading routers or using VLANs and other kinds of network isolation or "guest" networks.

See references below for more information.
- https://www.mbreviews.com/cascading-routers/
- https://www.michaelhorowitz.com/second.router.for.wfh.php
- https://about.gitlab.com/handbook/security/network-isolation/
- https://www.routersecurity.org/vlan.php

**Validator Security AMA**
- https://www.youtube.com/watch?v=o3V052VvI4o

References
- https://www.youtube.com/watch?v=hHtvCGlPz-o
- https://www.coincashew.com/coins/overview-eth/guide-or-how-to-setup-a-validator-on-eth2-mainnet/part-i-installation/guide-or-security-best-practices-for-a-eth2-validator-beaconchain-node

# Networking
There are ports that need to be exposed to the Internet for your validator to operate, specially for Geth and Lighthouse it's TCP/UDP ports 30303 and 9000 respectively. There are two common ways to control the firewall on your network: the Linux server and the network (such as your router or gateway to the Internet).

## Server
On the Linux server, you can open ports like this (as seen in the code).

```
# firewall rules to allow go-pulse and lighthouse services
sudo ufw allow 30303/tcp
sudo ufw allow 30303/udp
sudo ufw allow 9000/tcp
sudo ufw allow 9000/udp
```

## Home Router
This depends on your router device and model, so you'll need to research how to open ports on your specific networking device.

## AWS Cloud
**Security groups** (firewall)
- TCP port 22 (SSH) is your remote access to the server in the cloud (it’s enabled by default)
- For Erigon (or Geth), we need to open up TCP ports 30303 and 42069
- For Prysm (or Lighthouse), we need to open up TCP ports 9000 and 13000 as well as UDP port 12000
- All with Source=0.0.0.0/24 or Anywhere (unless you want to restrict SSH access to your specific IP range, but that's out of scope here)

# Graffiti

You can add graffiti flags for simple, one-line messages or ask Lighthouse to read from a file to express yourself in text for blocks your validator helps create. By default, the script does not set graffiti flags, but you can do this manually by editing the Lighthouse service files and adding in the flags and values that you want.

Check out the Lighthouse manual page on [graffiti](https://lighthouse-book.sigmaprime.io/graffiti.html) for instructions on how it works.

**Example**
```
$ sudo pico /etc/systemd/system/lighthouse-beacon.service

add something like this to the ExecStart= command (line 12)

 --graffiti "richard heart was right"

$ sudo systemctl daemon-reload
$ sudo systemctl restart lighthouse-beacon
```

# Withdrawals
**These instructions are only meant for use on Testnet and have not been tested on Mainnet, so only use them on Testnet until further testing and confirmation.**

**Be EXTRA CAREFUL as mistakes here can cost you funds and you must use these instructions at your own risk and hold yourself fully accountable for control and actions with your own funds, just like in the other parts of crypto.**

There are **full withdrawals** and partial withdrawals. This section will focus on the full withdrawal and validator exit process.

## Overview

**If you set a withdrawal address** when generating your validator keys, you can check on the [launchpad withdrawal](https://launchpad.v4.testnet.pulsechain.com/en/withdrawals) page to verify withdrawals are enabled and then exit your validator (see process below).

**If you didn't set a withdrawal address** when generating your validator keys, you need to "upgrade your keys" (generate BLSToExecution JSON) using the staking deposit client and broadcast it via the Launchpad, **which as of now is unavailable**. Will update with further instructions as this feature to support the scenario becomes available. Then, you can exit your validator from the network.

## Withdrawal Keys

**TREAT THIS AS IF YOU ARE GENERATING YOUR VALIDATOR KEYS + SEED WORDS**

**PERFORM IT ON A DIFFERENT, SECURE MACHINE (not your validator server)**

Find the validator index for the specific validator you want to initiate an exit and withdrawal. Check on the beacon explorer with the validator’s public key (for example, 4369).

Download the latest staking deposit

```
$ git clone https://gitlab.com/pulsechaincom/staking-deposit-cli.git

$ ./deposit.sh generate-bls-to-execution-change

English

pulsechain-testnet-v4

(Enter seed words)

0

(Enter each validator index as currently shown on the network)

You can open your deposit JSON file and copy each validator’s public key into the beacon explorer one at a time to get each index, but they should be sequential, like 4369, 4370, 4371, etc OR just the launchpad as it will show you validator index and withdrawal_credentials.

(Enter each withdrawal_credentials for each validator which is also in the deposit JSON file)

(Enter your execution address, aka withdrawal address, the wallet you’ve securely, made sure you and only you have access to and want to send the deposited PLS from the network back into)

DOUBLE CONFIRM YOU HAVE ACCESS TO THIS WALLET ADDRESS

It cannot be changed and if you typo something here, TWICE, YOUR FUNDS WILL BE UNRECOVERABLE.

Once you’ve double checked, enter the address again
```

Now your bls_to_execution_change JSON file is in the newly created **bls_to_execution_changes** folder.

## Exiting

You can broadcast the change using your Lighthouse client (for the specific validator you want to exit and initiate withdrawal).

```
$ sudo -u node bash

$ /opt/lighthouse/lighthouse/lh account validator exit --network pulsechain_testnet_v4 --keystore ~/.lighthouse/pulsechain_testnet_v4/validators/0x…(the validator public key you want to exit)/keystore-(specific for your setup)...json

Enter the keystore password

Enter the exit phrase described @ https://lighthouse-book.sigmaprime.io/voluntary-exit.html

“Successfully validated and published voluntary exit for validator 0x...” – and we can check it’s status on the beacon explorer

"Waiting for voluntary exit to be accepted into the beacon chain..."

"Voluntary exit has been accepted into the beacon chain, but not yet finalized. Finalization may take several minutes or longer. Before finalization there is a low probability that the exit may be reverted."

https://beacon.v4.testnet.pulsechain.com/validator/0x...
```

And you can see it’s going from Active to Exit (pulsing green).

Once it's exited, you have to wait for Withdrawals to become available.

```
This validator has exited the system during epoch 5369 and is no longer validating.

There is no need to keep the validator running anymore. Funds will be withdrawable after epoch 5555. 
```

References
- https://lighthouse-book.sigmaprime.io/voluntary-exit.html
- https://finematics.com/ethereum-staking-withdrawals-explained
- https://blog.stake.fish/eth-withdrawals-for-validators-your-go-to-guide-after-shanghai/
- https://docs.prylabs.network/docs/wallet/withdraw-validator
- https://docs.prylabs.network/docs/wallet/exiting-a-validator
- https://www.coincashew.com/coins/overview-eth/update-withdrawal-keys-for-ethereum-validator-bls-to-execution-change-or-0x00-to-0x01-with-ethdo
- https://nimbus.guide/withdrawals.html
- https://someresat.medium.com/guide-to-configuring-withdrawal-credentials-on-ethereum-812dce3193a
- https://github.com/eth-educators/ethstaker-guides/blob/main/zhejiang.md#adding-a-withdrawal-address
- https://www.youtube.com/watch?v=RwwU3P9n3uo

# Backups

## Home
You can use various tools on Linux to make scheduled backups to another disk OR another server.

References
- https://helpdeskgeek.com/linux-tips/5-ways-to-automate-a-file-backup-in-linux/
- https://www.howtogeek.com/135533/how-to-use-rsync-to-backup-your-data-on-linux/
- https://averagelinuxuser.com/automatically-backup-linux/
- https://www.math.cmu.edu/~gautam/sj/blog/20200216-rsync-backups.html
- https://www.simplified.guide/linux/automatic-backup

## Cloud
**Snapshots**
- AWS Home
- EC2 -> Instances -> (Select validator server’s Instance ID)
- Storage tab (near bottom)
- Click the Volume ID (to filter by it)
- Click the Volume ID (again)
- Actions -> Create Snapshot
- Description: (current date, for example 5/5/55)
- Create Snapshot

You should see in green...
- Successfully created snapshot snap-aaaabbbb from volume vol-xxxxyyyy.

It will be in Pending status for a while before the process completes (could be a few hours).

**Using a Snapshot**
- EC2 -> Snapshots
- Click on the Snapshot ID (see description to identify the right one, set Name as appropriate)
- Now you can do things like create a volume from the snapshot (and use the snapshot)
- Create a new volume from the snapshot
- Go back to volumes and name it like pulsechain-testnet-v4-snapshot-050523
- Spin up another server with the same hardware
- Create a new server (instance)
- Go to the new instance and detach the initially created volume
- EC2 -> Volumes -> Select the volume created from the snapshot
- Actions -> Attach volume
- Select the new instance (just created)
- Device name: /dev/sda1
- Click Attach volume
- Now start the new instance

You now have a new server with a hard disk volume based on the snapshot of the other server, yay!

# FAQ

* What server specs do you need to be a validator?

Specs and preferences vary between who you talk to, but at least 32gb ram and a beefy i7 or server-based processor and 2TB SSD hard disk. In the cloud, this roughly translates into a M2.2XLarge EC2 instance + 2TB disk.

* How long does it take to sync the blockchain clients?

It depends on your bandwidth, server specs and the state of the network, but you should expect anywhere from 24 - 96hrs for a validator node to sync.

* Can I run more than (1) validator on a single server?

Yes, you can run as many validators as you want on your server. Only caveat being that if you plan to run 100+, you may want to double the specs (at least memory) on your hardware to account for any additional resource usage. If you plan on running 1 or 10 or even 50, the minimum recommended hardware specs will probably work.

The setup script has no dependencies on the number of validators you run, it simply installs the clients and when you generate your validator keys with the staking deposit tool, there you choose the specific number you want to run. It could be 1, 5, 10 or 100. Then, when you import your keys to Lighthouse, you will import each key and it will configure the client to run that number of validators.

* How can I see the stats on my validator(s)?

Look at your deposit JSON file to get the list of your validator(s) public keys, then check https://beacon.v4.testnet.pulsechain.com/validator/ + your validator's public key which each one that you want to check the stats on.

For example this validator's stats: https://beacon.v4.testnet.pulsechain.com/validator/8001503cd43190b01aaa444d966a41ddb95c140e4910bb00ad638a4c020bc3a070612f318e3372109f33e40e7c268b0b

* What if my validator stops working?

Did your server's IP address change? If so, update lighthouse beacon service file @ /etc/systemd/system/lighthouse-beacon.service.

Did your network/firewall role change? Make sure the required client ports are accessible.

What is your status on the beacon explorer? Active, Pending, Exited or something else? If not active, it may be a client issue which you can debug with the steps discussed in the Debugging section.

Are your clients fully synced? They must be synced, talking to each other and talking to the network for the validator to work properly.

* How much does it cost to be a validator?

Depends on if you're using your own hardware or the cloud. For example, you could build or buy your own hardware for initial cost of around $2k and then pay for electricity it uses from running 24/7 each month. Or you can rent a server in the Amazon AWS cloud for an estimated $300-$500 per month. Both ways have advantages and disadvantages.

* Where can I find additional help on PulseChain dev stuff and being a validator?

https://t.me/PulseDev

# Additional Resources and References
- https://gitlab.com/pulsechaincom
- https://gammadevops.substack.com/p/part-1-introduction-to-validator
- https://gitlab.com/davidfeder/validatorscript/-/blob/64f37685908a78c5337f8d3dc951f7f01f251697/PulseChain_V4_Script.txt
- https://gitlab.com/davidfeder/validatorscript/-/blob/5fa11c7f81d8292779774b8dff9144ec3e44d26a/PulseChain_V3_Script.txt
- https://www.hexpulse.info/docs/node-setup.html
- https://togosh.medium.com/pulsechain-validator-setup-guide-70edae00b344
- https://github.com/tdslaine/install_pulse_node
- https://gitlab.com/Gamesys10/pulsechain-node-guide
- https://lighthouse-book.sigmaprime.io/api-lighthouse.html
- https://lighthouse-book.sigmaprime.io/key-management.html
- https://docs.gnosischain.com/node/guide/validator/run/lighthouse
- https://ethereum.stackexchange.com/questions/394/how-can-i-find-out-what-the-highest-block-is
- https://www.coincashew.com/coins/overview-eth/guide-or-how-to-setup-a-validator-on-eth2-mainnet/part-i-installation/monitoring-your-validator-with-grafana-and-prometheus
- https://schh.medium.com/port-forwarding-via-ssh-ba8df700f34d
- https://www.youtube.com/watch?v=lbUnlIL_yLs&ab_channel=Oooly
- https://www.reddit.com/r/ethstaker/comments/txj5vh/technical_overview_of_validator_need_some_help/
- https://docs.prylabs.network/docs/troubleshooting/issues-errors
- https://pawelurbanek.com/ethereum-node-aws
- https://chasewright.com/getting-started-with-turbo-geth-on-ubuntu/
- https://docs.prylabs.network/docs/prysm-usage/p2p-host-ip
- https://www.blocknative.com/blog/an-ethereum-stakers-guide-to-slashing-other-penalties
- https://goerli.launchpad.ethstaker.cc/en/faq
- https://www.coincashew.com/coins/overview-eth/guide-or-how-to-setup-a-validator-on-eth2-mainnet/part-i-installation/guide-or-security-best-practices-for-a-eth2-validator-beaconchain-node
- https://ethereum.stackexchange.com/questions/3887/how-to-reduce-the-chances-of-your-ethereum-wallet-getting-hacked
- https://docs.prylabs.network/docs/install/install-with-script
- https://7goldfish.com/Eth_Staking_Testnet_on_AWS.html
- https://mirror.xyz/steinkirch.eth/F5PI4eqShKTGlx0GzL0Lq0-vHQ6b14OoV4ylE2FMsAc
- https://consensys.net/blog/developers/my-journey-to-being-a-validator-on-ethereum-2-0-part-5/
- https://www.monkeyvault.net/secure-aws-infrastructure-with-vpc-a-terraform-guide/ (VPCs guide too)
- https://hackmd.io/@prysmaticlabs/HkSSMpDtt
- https://medium.com/@mshmulevich/running-ethereum-nodes-in-high-availability-cluster-on-aws-aefd08d4d81
- https://chasewright.com/getting-started-with-turbo-geth-on-ubuntu/
- https://someresat.medium.com/guide-to-staking-on-ethereum-ubuntu-prysm-581fb1969460
- https://www.blocknative.com/blog/ethereum-validator-lighthouse-geth
- https://www.youtube.com/watch?v=hHtvCGlPz-o
