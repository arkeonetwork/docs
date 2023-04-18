# Setting Up a Validator for Arkeo

This document will guide you through the process of setting up a validator for Arkeo. Validators are responsible for validating and committing blocks in a Proof-of-Stake (PoS) blockchain network.

## Prerequisites

Before you start setting up a validator, make sure you have the following software installed:

- Go 1.20 or higher

Also, ensure that you have sufficient tokens to stake as a validator.

## Steps to Set Up a Validator

### 1. Install and build the project

Clone the arkeo repository and build the CLI and daemon binaries:

```bash
git clone https://github.com/arkeonetwork/arkeo.git
cd arkeo
make install

### 2. Initialize the node

Initialize your node using the `arkeod init` command. Replace `<your_moniker>` with a name for your node.

```bash
arkeod init <your_moniker> --chain-id <your_chain_id>
```

You can get the `chain_id` from any currently available validators
```bash
curl -sL <ip address>:<port>/status | jq .result.node_info.network
```

### 3. Generate a new key pair for the validator

Generate a new key pair for your validator using the following command:

```bash
arkeod keys add <your_validator_key_name>
```

**Important: Make sure to securely store the mnemonic phrase, as it is required for key recovery.**


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
