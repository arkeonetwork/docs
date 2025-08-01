# Setup for Data Providers

This guide walks you through setting up your Arkeo Provider—from bonding your service on-chain, to configuring metadata, running your backend (sentinel), and automating settlement. You do not need to be a validator or run a full node to be a provider, though running your own node is strongly recommended for reliability.

## Before You Begin

### What You Need

- **Access to a synced Arkeo Node**
  - Host your own. (Recommended)
  - Connect with someone who has their ports open.
- **Arkeo Accounts:**
  - **Hot Wallet** (automated settlement key created with keyring-backend test).
  - (Optional) Secure cold storage account for larger funds.
- **Arkeo CLI Tools** installed (from GitHub).
- **Server** or reliable machine for running your backend (sentinel).
- **Recommended:** A full node for complete control over blockchain events and reliable settlements.

## Install Arkeo Tools

```
git clone https://github.com/arkeonetwork/arkeo.git  
cd arkeo  
make install  
make tools
```

This installs the arkeod CLI and related tools.

> **Best Practice:**  
> Although no full chain sync is required for basic operations, running your own node is highly recommended. Doing so ensures full control over blockchain events critical for timely settlements, avoiding reliance on third-party RPC services.

## Syncing or Accessing an Arkeo Node

For basic testing and setup, I think it's fine to use my provider, as the ports are open.

Some command line examples:
```
arkeod status --node tcp://provider1.innovationtheory.com:26657

ARKEOD_NODE="tcp://provider1.innovationtheory.com:26657" arkeod query arkeo all-services
```

Or the api reference
```
http://provider1.innovationtheory.com:1317/arkeo/services
```
> **Best Practice:**  
> If you want to do anything at a production level, you will want to have a synced arkeo Daemon running connected live to the Arkeo blockchain.

Follow the setup instructions:
- [Running a Node](https://github.com/arkeonetwork/arkeo?tab=readme-ov-file)
- [Validator Instructions](../validators/validators.md)

Then you can use the most recent snapshot as a shortcut to syncing.

```
wget http://seed.innovationtheory.com:8080/arkeo_snapshot.tar.gz
```
Either way, make sure you have access to a synced Arkeo node before moving forward.

## Create a Hot Wallet (Automated Key)

Providers automate claim settlements using a dedicated "hot wallet":

```
arkeod keys add <provider-hot-wallet> --keyring-backend test
```

- Fund this wallet only with minimal amounts required for bonding and fees.

> Security Notice:
> - The test backend stores your keys unencrypted; use it only for automated scripts.
> - Never use this wallet for significant funds.

## Arkeo Supported Services

Use either of these services for a comprehensive reference mapping between service names and their numeric identifiers, as used within Arkeo.

```
arkeod query arkeo all-services
```
```
http://localhost:1317/arkeo/services
```

## Bond Your Provider Service On-Chain
Bonding stakes collateral and registers your service.

> No Validator privileges are required to be a Provider.
>
> Each time you run the BOND script, you will add in more tokens- so be aware that the bond will keep increasing.
>
> Just add a negative bond amount to UNBOND tokens, and UNBOND all tokens to remove your provider.
>
> The Bond is implemented so that providers have skin in the game and can be docked for bad behavior or performance. This functionality will be worked into the system soon.

Usage:
```
./provider_bond_4.sh arkeo-mainnet-fullnode
```
```
#!/bin/bash

if [ -z "$1" ]; then
  echo "Usage: $0 <SERVICE>"
  exit 1
fi
SERVICE="$1"

USER="Arkeo-Main-Validator-1"
BOND=100000000
KEYRING_BACKEND="test"
FEES="200uarkeo"

# Get raw public key
RAWPUBKEY=$(arkeod keys show "$USER" -p --keyring-backend="$KEYRING_BACKEND" | jq -r .key)

# Convert to Bech32 pubkey
PUBKEY=$(arkeod debug pubkey-raw $RAWPUBKEY | grep 'Bech32 Acc:' | sed "s|Bech32 Acc: ||g")

#echo "Using USER: $USER"
#echo "RAW PUBKEY: $RAWPUBKEY"
#echo "PUBKEY: $PUBKEY"
#echo "SERVICE: $SERVICE"
#echo "KEYRING_BACKEND: $KEYRING_BACKEND"
#echo "FEES: $FEES"
#echo "BOND: $BOND"

echo "Bonding provider service '$SERVICE'..."

# Bond the provider
arkeod tx arkeo bond-provider \
  --from="$USER" --fees="$FEES" --keyring-backend="$KEYRING_BACKEND" -b sync -y \
  "$PUBKEY" "$SERVICE" -- "$BOND"

if [ $? -ne 0 ]; then
  echo "Error: Provider bonding transaction failed."
  exit 1
else
  echo "Provider bonding transaction submitted successfully."
fi
```

## Configure Provider Metadata
Make your service discoverable and configure contract terms:

> The Nonce is cosmetic, and not required to be incremented like in Contracts.

Usage:
```
./provider_mod_4.sh arkeo-mainnet-fullnode 1 1 200uarkeo 200uarkeo
```

```
#!/bin/bash

# Usage check for required arguments
if [ "$#" -lt 5 ]; then
  echo "Usage: $0 <SERVICE> <METADATA_NONCE> <STATUS> <SUBSCRIPTION_RATES> <PAY_AS_YOU_GO_RATES>"
  exit 1
fi
SERVICE="$1"
METADATA_NONCE="$2"
STATUS="$3"
SUBSCRIPTION_RATES="$4"
PAY_AS_YOU_GO_RATES="$5"

KEY="Arkeo-Main-Validator-1"
SENTINEL_URI="http://127.0.0.1:3636/metadata.json"
MIN_CONTRACT_DUR=5
MAX_CONTRACT_DUR=432000
SETTLEMENT_DUR=100
KEYRING_BACKEND="test"
FEES="200uarkeo"

# Get raw public key
RAWPUBKEY=$(arkeod keys show "$KEY" -p --keyring-backend="$KEYRING_BACKEND" | jq -r .key)

# Convert to Bech32 pubkey
PUBKEY=$(arkeod debug pubkey-raw $RAWPUBKEY | grep 'Bech32 Acc:' | sed "s|Bech32 Acc: ||g")

echo "Modifying provider for service '$SERVICE'..."

# Mod the provider
arkeod tx arkeo mod-provider \
"$PUBKEY" \
"$SERVICE" \
"$SENTINEL_URI" \
"$METADATA_NONCE" \
"$STATUS" \
"$MIN_CONTRACT_DUR" \
"$MAX_CONTRACT_DUR" \
"$SUBSCRIPTION_RATES" \
"$PAY_AS_YOU_GO_RATES" \
"$SETTLEMENT_DUR" \
--from="$KEY" \
--fees="$FEES" \
--keyring-backend="$KEYRING_BACKEND" \
-y

if [ $? -ne 0 ]; then
  echo "Error: Provider modification transaction failed."
  exit 1
else
  echo "Provider modification transaction submitted successfully."
fi
```

## Provider Bond + Mod Looping Add/Update Script
This is a fast onboarding script to setup multiple providers at once, quickly.

```
#!/bin/bash

# Paths to your scripts (edit if needed)
BOND_SCRIPT="./provider_bond_4.sh"
MOD_SCRIPT="./provider_mod_4.sh"
SERVICES_API_URL="http://localhost:1317/arkeo/services"

# SERVICE_TYPES contains lines of: SERVICE METADATA_NONCE STATUS SUBSCRIPTION_RATES PAY_AS_YOU_GO_RATES
# Example:
#   "akash-mainnet-fullnode 1 1 200uarkeo 200uarkeo"
# Where:
#   SERVICE            = the service type string
#   METADATA_NONCE     = integer metadata nonce for this service
#   STATUS             = 1 (ONLINE) or 0 (OFFLINE)
#   SUBSCRIPTION_RATES = e.g., 200uarkeo
#   PAY_AS_YOU_GO_RATES= e.g., 200uarkeo

SERVICE_TYPES=(
  "btc-mainnet-fullnode 1 1 200uarkeo 200uarkeo"
)

for SERVICE_ENTRY in "${SERVICE_TYPES[@]}"; do
  set -- $SERVICE_ENTRY
  SERVICE="$1"
  METADATA_NONCE="$2"
  STATUS="$3"
  SUBSCRIPTION_RATES="$4"
  PAY_AS_YOU_GO_RATES="$5"

  echo ""

  BOND_OUTPUT=$($BOND_SCRIPT "$SERVICE")
  echo "$BOND_OUTPUT"
  echo ""
  BOND_TXHASH=$(echo "$BOND_OUTPUT" | grep "txhash:" | awk '{print $2}')
  if [ -n "$BOND_TXHASH" ]; then
    sleep 6
    echo "Querying bond transaction $BOND_TXHASH ..."
    arkeod query tx $BOND_TXHASH
  else
    echo "No transaction hash found for bond."
  fi
  if [ $? -ne 0 ]; then
    echo "Bonding failed for $SERVICE. Skipping modification."
    continue
  fi

  echo ""

  MOD_OUTPUT=$($MOD_SCRIPT "$SERVICE" $METADATA_NONCE $STATUS "$SUBSCRIPTION_RATES" "$PAY_AS_YOU_GO_RATES")
  echo "$MOD_OUTPUT"
  echo ""
  MOD_TXHASH=$(echo "$MOD_OUTPUT" | grep "txhash:" | awk '{print $2}')
  if [ -n "$MOD_TXHASH" ]; then
    sleep 6
    echo "Querying mod transaction $MOD_TXHASH ..."
    arkeod query tx $MOD_TXHASH
  else
    echo "No transaction hash found for mod."
  fi
  if [ $? -ne 0 ]; then
    echo "Modification failed for $SERVICE."
  fi

done

SERVICES_JSON=$(curl -s $SERVICES_API_URL)

# Now output YAML config block for all service types
echo ""
echo "services:"
for SERVICE_ENTRY in "${SERVICE_TYPES[@]}"; do
  set -- $SERVICE_ENTRY
  SERVICE="$1"
  CHAIN_TYPE=$(echo "$SERVICE" | cut -d'-' -f1)
  SERVICE_ID=$(echo "$SERVICES_JSON" | jq ".services[] | select(.name==\"$SERVICE\") | .service_id")
  echo "  - name: $SERVICE"
  echo "    id: $SERVICE_ID"
  echo "    type: $CHAIN_TYPE"
  echo "    rpc_url: http://127.0.0.1:26657"
  echo "    rpc_user:"
  echo "    rpc_pass:"
done
```

## Provider and Sentinel Configuration

Example sentinel_config.yaml:

```
provider:
pubkey: "<your-bech32-pubkey>"
name: "My Arkeo Provider"

services:
- name: btc-mainnet-fullnode
  id: 10
  type: bitcoin
  rpc_url: http://localhost:8332
  rpc_user: "<rpc-username>"
  rpc_pass: "<rpc-password>"

api:
listen_addr: "0.0.0.0:3636"
```

## Run Sentinel

Create and enable sentinel.service with systemd:

```
[Unit]
Description=Arkeo Sentinel (Provider)
After=network-online.target

[Service]
User=<your-os-user>
WorkingDirectory=/home/<your-os-user>
ExecStart=/home/<your-os-user>/go/bin/sentinel --config /home/<your-os-user>/sentinel_config.yaml
Restart=on-failure
RestartSec=5
LimitNOFILE=65535
Environment="CLAIM_STORE_LOCATION=/home/<your-os-user>/.arkeo/claims"
Environment="CONTRACT_CONFIG_STORE_LOCATION=/home/<your-os-user>/.arkeo/contract_configs"
Environment="DESCRIPTION=Arkeo Core Multi-Provider: Core Provider of Blockchain Nodes and Indexers."
Environment="EVENT_STREAM_HOST=127.0.0.1:26657"
Environment="FREE_RATE_LIMIT=10"
Environment="LOCATION=USA, West Coast"
Environment="LOG_LEVEL=debug"
Environment="MONIKER=arkeo-core-multi-provider"
Environment="PORT=3636"
Environment="PROVIDER_CONFIG_STORE_LOCATION=/home/<your-os-user>/.arkeo/provider_configs"
Environment="PROVIDER_HUB_URI=http://127.0.0.1:1317"
Environment="PROVIDER_PUBKEY=<your-bech32-pubkey>"
Environment="SOURCE_CHAIN=arkeo-main-v2"
Environment="WEBSITE=https://arkeo.network"

[Install]
WantedBy=multi-user.target
```

Activate the service:

```
sudo systemctl daemon-reload
sudo systemctl enable --now sentinel
```

## Automate Settlements

Set up your settlement script to use the "hot wallet" (keyring-backend test) for automated signing:
- Do not store hot wallet mnemonics in plain text or public repositories.
- Regularly monitor hot wallet transactions and balances.

## Monitoring and Verification

Check provider status:
```
arkeod query arkeo show-provider <your-bech32-pubkey>
```
Metadata API:
```
curl http://localhost:3636/metadata.json
```
Logs:
```
journalctl -u sentinel -f
```

## Additional Recommendations

- **Run your own full node** for reliable and timely event processing.
- Limit reliance on third-party RPC providers to maintain control over settlements.
- Rotate and audit hot wallet keys regularly.