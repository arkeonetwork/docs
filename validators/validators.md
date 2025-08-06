# Validator Setup

Validators play a critical role in securing and maintaining the Arkeo blockchain. This concise guide outlines the essential steps to become and manage a validator.

## Prerequisites

- A secure, dedicated server with reliable uptime.
- Installed and synced arkeod full node.
- An Arkeo account funded with the required minimum staking tokens.

## Step-by-Step Setup

### Install Arkeo Node

Follow official guidelines to install and sync a full Arkeo node:

```
git clone https://github.com/arkeonetwork/arkeo.git
cd arkeo
make install
```

### Install Cosmovisor

**Build Cosmovisor**
```bash
go install github.com/cosmos/cosmos-sdk/cosmovisor/cmd/cosmovisor@v1.0.0
```

**Place Cosmovisor in /usr/local/bin**
```
sudo cp $(which cosmovisor) /usr/local/bin/
```

### Create or Import Validator Key
```
arkeod keys add <validator-key-name> --keyring-backend file
```

### Prepare Cosmovisor Directory Structure
Assuming your Arkeo home is ~/.arkeo (default):

```
mkdir -p ~/.arkeo/cosmovisor/genesis/bin
mkdir -p ~/.arkeo/cosmovisor/upgrades
```

Copy the current arkeod binary to the genesis bin:
```
cp ~/go/bin/arkeod ~/cosmovisor/genesis/bin/
```


### Update Service to Use Cosmovisor
Create a systemd service for Cosmovisor:

```
# /etc/systemd/system/arkeod.service

[Unit]
Description=Arkeo Node (Cosmovisor)
After=network-online.target

[Service]
User=<ARKEO_USER>
ExecStart=/usr/local/bin/cosmovisor run start
Restart=always
RestartSec=10
LimitNOFILE=4096
Environment="DAEMON_NAME=arkeod"
Environment="DAEMON_HOME=/home/<ARKEO_USER>/.arkeo"
Environment="DAEMON_ALLOW_DOWNLOAD_BINARIES=false"
Environment="DAEMON_RESTART_AFTER_UPGRADE=true"

[Install]
WantedBy=multi-user.target
```


### Fund Validator Account

Ensure your validator account is funded sufficiently for staking and fees.

## Validator Creation

Create your validator on-chain:

```
arkeod tx staking create-validator \
--amount 1000000uarkeo \
--pubkey $(arkeod tendermint show-validator) \
--moniker "YourValidatorName" \
--chain-id arkeo-main-v1 \
--commission-rate "0.10" \
--commission-max-rate "0.20" \
--commission-max-change-rate "0.01" \
--min-self-delegation "1" \
--from <validator-key-name> \
--keyring-backend file \
--fees 200uarkeo \
-y
```

Adjust values as needed.

## Validator Management Commands

- Validator status:
```
arkeod query staking validator <validator-address>
```
- Delegate more tokens:
```
arkeod tx staking delegate <validator-address> 500000uarkeo --from <validator-key-name> --fees 200uarkeo -y
```
- Withdraw validator rewards:
```
arkeod tx distribution withdraw-rewards <validator-address> --from <validator-key-name> --fees 200uarkeo -y
```

## Cosmovisor Upgrade Flow
For chain upgrades, place the new arkeod binary under:

```
~/.arkeo/cosmovisor/upgrades/<UpgradeName>/bin/arkeod
```
Cosmovisor will auto-switch at the block height specified by the chain upgrade proposal.



## Best Practices

- Regularly monitor validator health and logs.
- Maintain sufficient funds to cover transaction fees and maintain stake.
- Keep your validator node updated and secured.
