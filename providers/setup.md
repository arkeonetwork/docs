# Setup for Data Providers

This guide walks you through setting up your Arkeo Provider—from bonding your service on-chain, to configuring metadata, running your backend (sentinel), and automating settlement. You do not need to be a validator or run a full node to be a provider, though running your own node is strongly recommended for reliability.

## Before You Begin

### What You Need

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

## Create a Hot Wallet (Automated Key)

Providers automate claim settlements using a dedicated "hot wallet":

```
arkeod keys add <provider-hot-wallet> --keyring-backend test
```

- Fund this wallet only with minimal amounts required for bonding and fees.

> Security Notice:
> - The test backend stores your keys unencrypted; use it only for automated scripts.
> - Never use this wallet for significant funds.

## Bond Your Provider Service On-Chain
Bonding stakes collateral and registers your service:

```
USER="<provider-hot-wallet>"
SERVICE="<your-service-name>" 
BOND=100000000 
KEYRING_BACKEND="test"
FEES="200uarkeo"

RAWPUBKEY=$(arkeod keys show "$USER" -p --keyring-backend="$KEYRING_BACKEND" | jq -r .key)
PUBKEY=$(arkeod debug pubkey-raw $RAWPUBKEY | grep 'Bech32 Acc:' | sed "s|Bech32 Acc: ||g")

arkeod tx arkeo bond-provider "$PUBKEY" "$SERVICE" "$BOND" \
--from="$USER" \
--fees="$FEES" \
--keyring-backend="$KEYRING_BACKEND" \
-b sync \
-y
```

> Notice:
> - No Validator privileges are required to be a Provider.

## Configure Provider Metadata
Make your service discoverable and configure contract terms:

```
SENTINEL_URI="http://<your-domain-or-ip>:3636/metadata.json"
METADATA_NONCE=1
STATUS=1
MIN_CONTRACT_DUR=5
MAX_CONTRACT_DUR=432000
SUBSCRIPTION_RATES="200uarkeo"
PAY_AS_YOU_GO_RATES="200uarkeo"
SETTLEMENT_DUR=1000

arkeod tx arkeo mod-provider \
"$PUBKEY" "$SERVICE" "$SENTINEL_URI" "$METADATA_NONCE" "$STATUS" \
"$MIN_CONTRACT_DUR" "$MAX_CONTRACT_DUR" "$SUBSCRIPTION_RATES" \
"$PAY_AS_YOU_GO_RATES" "$SETTLEMENT_DUR" \
--from="$USER" --fees="$FEES" \
--keyring-backend="$KEYRING_BACKEND" -y
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