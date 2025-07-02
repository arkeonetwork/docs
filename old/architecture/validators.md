# Setting Up a Validator for Arkeo

This document provides a step-by-step guide to setting up a validator for the Arkeo blockchain. Validators play a crucial role in Proof-of-Stake (PoS) networks by validating and committing blocks. This process involves running nodes and staking Arkeo tokens to actively participate in the network's consensus mechanism.


## Prerequisites

Before you start setting up a validator, make sure you have the following software installed:

- Go 1.21 or higher

Also, ensure that you have sufficient tokens to stake as a validator.

## Steps to Set Up a Validator

### 1. Install and build the project

Option 1: Build from Source

```bash
git clone https://github.com/arkeonetwork/arkeo.git
cd arkeo
make install
```
Option 2: Use Prebuilt Release Binaries
```bash
VERSION=$(curl -s https://api.github.com/repos/arkeonetwork/arkeo/releases/latest | grep tag_name | cut -d '"' -f 4)
BINARY_URL="https://github.com/arkeonetwork/arkeo/releases/download/$VERSION/<binary-name>.tar.gz"
curl -L $BINARY_URL -o arkeod.tar.gz
mkdir -p $HOME/go/bin
tar -xzf arkeod.tar.gz -C $HOME/go/bin
chmod +x $HOME/go/bin/arkeod
rm arkeod.tar.gz
```


### 2. Generate a new key pair for the validator and configure node binary

Generate a new key pair for your validator using the following command:

```bash
arkeod keys add <your_validator_key_name> 
```

**Important: Make sure to securely store the mnemonic phrase, as it is required for key recovery.**

You can get the `chain_id` from any currently available validators
```bash
curl -sL <ip address>:<port>/status | jq .result.node_info.network
```

Configure node binary using below commands:
```bash
arkeod config set client chain-id <your_chain_id>
arkeod config set client keyring-backend <keyring>
```

Download genesis file 
```bash
curl -s <ip address>:<port>/genesis | jq '.result.genesis' > $HOME/.arkeo/config/genesis.json
```


Configure App Setting: Pruning, Gas Prices, Swagger and Prometheus
```bash
sed -i -e "s/^pruning *=.*/pruning = \"custom\"/" $HOME/.arkeo/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"100\"/" $HOME/.arkeo/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"50\"/" $HOME/.arkeo/config/app.toml
sed -i 's|minimum-gas-prices =.*|minimum-gas-prices = "<minimum-price>uarkeo"|g' $HOME/.arkeo/config/app.toml
sed -i 's|swagger =.*| swagger = true|g' $HOME/.arkeo/config/app.toml
```

Configure Seeds and Peers: Set Seeds and Peers
```bash
SEEDS="<seed goes here>"
PEERS="peers comma separated"
sed -i -e "/^\[p2p\]/,/^\[/{s/^[[:space:]]*seeds *=.*/seeds = \"$SEEDS\"/}" \
  -e "/^\[p2p\]/,/^\[/{s/^[[:space:]]*persistent_peers *=.*/persistent_peers = \"$PEERS\"/}" $HOME/.arkeo/config/config.toml
```

### 3. Initialize the node

Initialize your node using the `arkeod init` command. Replace `<your_moniker>` with a name for your node.

```bash
arkeod init <your_moniker> --chain-id <your_chain_id>
```

### 4. Create the validator

Now, create the validator by running the following command:

```bash
arkeod tx staking create-validator \
  --amount=<your_staking_amount>uarkeo \
  --pubkey=$(arkeod tendermint show-validator) \
  --moniker=<your_moniker> \
  --chain-id=<your_chain_id> \
  --commission-rate="0.10" \
  --commission-max-rate="0.20" \
  --commission-max-change-rate="0.01" \
  --min-self-delegation="1" \
  --gas="auto" \
  --gas-prices="<your_gas_price>uarkeo" \
  --from=<your_validator_key_name>
```

Replace **<your_staking_amount>**, **<your_token_denom>**, **<your_gas_price>**, and **<your_validator_key_name>** with appropriate values. This command will broadcast a transaction to create the validator.

### 5. Start the node

After creating the validator, start the node with the following command:

```bash
arkeod start
```

Your node will now sync with the network and begin validating blocks. Congratulations, you have successfully set up a validator for Arkeo!

## Additional Resources

For more information on validators and Cosmos SDK, please refer to the following resources:

- [Cosmos Validators](https://docs.cosmos.network/master/validators/overview.html)
- [Cosmos SDK Documentation](https://docs.cosmos.network/master/)
- [Cosmos SDK GitHub Repository](https://github.com/cosmos/cosmos-sdk)
