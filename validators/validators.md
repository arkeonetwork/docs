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

### Create or Import Validator Key

arkeod keys add <validator-key-name> --keyring-backend file

### Fund Validator Account

Ensure your validator account is funded sufficiently for staking and fees.

## Validator Creation

Create your validator on-chain:

```
arkeod tx staking create-validator \
--amount 1000000uarkeo \
--pubkey $(arkeod tendermint show-validator) \
--moniker "YourValidatorName" \
--chain-id arkeo-main-v2 \
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

## Best Practices

- Regularly monitor validator health and logs.
- Maintain sufficient funds to cover transaction fees and maintain stake.
- Keep your validator node updated and secured.
