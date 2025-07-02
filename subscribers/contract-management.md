# Subscriber Contract Management

Subscribers have flexibility in managing their contracts to ensure continuous uptime. The examples provided here illustrate a strategy using a single provider with multiple Pay-As-You-Go (PAYG) contracts, where the most recent active contract is always utilized. Subscribers may also apply similar logic to subscription contracts by automating contract renewal or implementing failover mechanisms upon expiry. The autocaller scripts demonstrated here are intelligent tools designed to reliably select the latest active contract for your chosen provider and service, ensuring consistent application availability.

## Contract Management: PAYG Auto-Topper Script

This auto-topper script ensures continuous uptime for your Pay-As-You-Go (PAYG) contracts by automatically monitoring their usage and duration, and proactively creating new overlapping contracts when active contracts reach a specified threshold. This automated approach ensures that your service access remains uninterrupted.

```
#!/bin/bash

#
# CONFIGURATION:
# These are the configuration values used for each service to maintain the 100% uptime capabilities of the script.
#
# SERVICE_NAME:             Human-readable name of the blockchain or API service (e.g., "btc-mainnet-fullnode").
# SERVICE_NUMBER:           Value for the service name, pulled from the enum here: https://raw.githubusercontent.com/arkeonetwork/arkeo/refs/heads/master/common/service.go
# SERVICE_ARKEO_API:        API endpoint for the local Arkeo node (used for creating/querying contracts, e.g., http://127.0.0.1:26657).
# SERVICE_ARKEO_FEE:        Default fee to use when creating or updating contracts via Arkeo transactions (e.g., "200uarkeo").
# PROVIDER_PUBKEY:          Bech32 public key of the provider (the service operator who receives contract income).
# PROVIDER_SENTINEL_API:    API endpoint for the provider's Sentinel proxy (where client requests are sent for metering/billing).
# CLIENT_KEY:               Local key name in your keyring for the client wallet (the party opening/contracts).
# CLIENT_KEYRING:           Keyring backend for the client key ("test", "file", "os", etc.).
# CONTRACT_TYPE:            Type of contract: 0 = subscription (fixed time), 1 = pay-as-you-go (usage-based).
# CONTRACT_AUTH:            Authorization model: 0 = STRICT (per-request client signatures), 1 = OPEN (no signatures required).
# CONTRACT_DEPOSIT:         Total amount of tokens (in smallest denom) to lock in the contract (e.g., "100000000" for 1 ARKEO).
# CONTRACT_DURATION:        Duration of contract in blocks (for subscriptions), or can be short for PAYG.
# CONTRACT_RATE:            Cost per request (matches provider's advertised rate, in smallest denom).
# CONTRACT_QPM:             Maximum queries per minute allowed under the contract (for rate limiting).
# CONTRACT_SETTLEMENT:      Number of blocks after contract expiry in which the provider can still submit claims.
# CONTRACT_DELEGATE:        (Optional) Bech32 pubkey of a delegate permitted to spend or claim from this contract.
#

CHAIN_ID="arkeo-main-v2"
CLIENT_KEY="Arkeo-Main-Validator-3"
CLIENT_KEYRING="test"
ARKEO_SERVICE_API="127.0.0.1:26657"
ARKEO_SERVICE_FEE="200uarkeo"
CLIENT_RAW_PUBKEY=$(arkeod keys show "$CLIENT_KEY" --output json --keyring-backend="$CLIENT_KEYRING" | jq -r .pubkey | jq -r .key)
CLIENT_PUBKEY=$(arkeod debug pubkey-raw "$CLIENT_RAW_PUBKEY" -t secp256k1 | grep 'Bech32 Acc:' | sed "s|Bech32 Acc: ||g")
CONTRACT_OVERLAP_THRESHOLD=20

SERVICES_LIST=(
'{
"SERVICE_NAME":             "btc-mainnet-fullnode",
"SERVICE_NUMBER":           10,
"PROVIDER_PUBKEY":          "arkeopub1addwnpepqfn52r6xng2wwfrgz2tm5yvscq42k3yu3ky9cg3kw5s6p0qg7tfx75uwq3z",
"PROVIDER_SENTINEL_API":    "http://127.0.0.1:3636",
"CONTRACT_TYPE":            1,
"CONTRACT_AUTH":            0,
"CONTRACT_DEPOSIT":         100000000,
"CONTRACT_DURATION":        100,
"CONTRACT_RATE":            "10000uarkeo",
"CONTRACT_QPM":             20,
"CONTRACT_SETTLEMENT":      100,
"CONTRACT_DELEGATE":        ""
}'
)

#
# OPEN ARKEO CONTRACT:
# This function is called when there are no contracts found and to create a second overlapping contract when the active contract reached 25%.
#

open_arkeo_contract() {

# Open the contract and capture the txhash
OPEN_CONTRACT_RESULT=$(arkeod tx arkeo open-contract \
"$PROVIDER_PUBKEY" \
"$SERVICE_NAME" \
"$CLIENT_PUBKEY" \
"$CONTRACT_TYPE" \
"$CONTRACT_DEPOSIT" \
"$CONTRACT_DURATION" \
"$CONTRACT_RATE" \
"$CONTRACT_QPM" \
"$CONTRACT_SETTLEMENT" \
"$CONTRACT_AUTH" \
"$CONTRACT_DELEGATE" \
--from="$CLIENT_KEY" \
--keyring-backend="$CLIENT_KEYRING" \
--fees="$ARKEO_SERVICE_FEE" \
--node tcp://$ARKEO_SERVICE_API \
-y --output json)

TXHASH=$(echo "$OPEN_CONTRACT_RESULT" | jq -r '.txhash // .TxHash // empty')
if [ -z "$TXHASH" ] || [ "$TXHASH" == "null" ]; then
echo "Failed to get txhash! Raw result:"
echo "$OPEN_CONTRACT_RESULT"
exit 1
fi
echo "TxHash: $TXHASH"

# Wait for transaction to be committed, retry if not present yet
TX_RESULT=""
for i in {1..10}; do
TX_RESULT=$(arkeod query tx "$TXHASH" --output json 2>/dev/null)
if echo "$TX_RESULT" | jq . > /dev/null 2>&1; then
break
fi
sleep 2
done

CONTRACT_ID=$(echo "$TX_RESULT" | jq -r '
.events[]?
| select(.type == "arkeo.arkeo.EventOpenContract")
| .attributes[]?
| select(.key == "contract_id")
| .value
' | tr -d '"')

if [ -z "$CONTRACT_ID" ]; then
echo "Could not extract contract id. Raw tx result:"
echo "$TX_RESULT" | jq
exit 1
fi

echo "Contract ID: $CONTRACT_ID"

# Sign POST request (update config)
NONCE_DATE=$(date +%s)
MSG_POST="$CONTRACT_ID:$NONCE_DATE:"
SIG_POST=$(signhere -u "$CLIENT_KEY" -m "$MSG_POST" | tail -n 1)
ARKAUTH_POST="$CONTRACT_ID:$NONCE_DATE:$SIG_POST"
API_URL_POST="$PROVIDER_SENTINEL_API/manage/contract/$CONTRACT_ID?arkcontract=$ARKAUTH_POST"

echo "POST URL: $API_URL_POST"

# Prepare the whitelist update JSON
read -r -d '' WHITELIST_JSON <<EOF
{
"white_listed_ip_addresses": [],
"per_user_rate_limit": 1000,
"cors": {
"allow_origins": ["*"],
"allow_methods": ["GET", "POST", "PUT", "DELETE", "OPTIONS"],
"allow_headers": ["*"]
}
}
EOF

echo "Whitelist: $WHITELIST_JSON"
echo "API Url: $API_URL_POST"

# Config contract data
curl -X POST -H "Content-Type: application/json" -d "$WHITELIST_JSON" "$API_URL_POST"

}

#
# SERVICE LOOP:
# Main loop to iterate over each service and manage contracts accordingly
#

echo
echo "Arkeo Pay-As-You-Go Contract Topper"
echo "  $(date)"

for SERVICE in "${SERVICES_LIST[@]}"; do

#
# SERVICE VARIABLE MAPPING
# Map the configuration values to variables.
#

SERVICE_NAME=$(echo "$SERVICE" | jq -r '.service_name // .SERVICE_NAME // empty')
SERVICE_NUM=$(echo "$SERVICE" | jq -r '.service_number // .SERVICE_NUMBER // empty')
SERVICE_ARKEO_API=$(echo "$SERVICE" | jq -r '.service_arkeo_api // .SERVICE_ARKEO_API // empty')
SERVICE_ARKEO_FEE=$(echo "$SERVICE" | jq -r '.service_arkeo_fee // .SERVICE_ARKEO_FEE // empty')
PROVIDER_PUBKEY=$(echo "$SERVICE" | jq -r '.provider_pubkey // .PROVIDER_PUBKEY // empty')
PROVIDER_SENTINEL_API=$(echo "$SERVICE" | jq -r '.provider_sentinel_api // .PROVIDER_SENTINEL_API // empty')
CONTRACT_TYPE=$(echo "$SERVICE" | jq -r '.contract_type // .CONTRACT_TYPE // empty')
CONTRACT_AUTH=$(echo "$SERVICE" | jq -r '.contract_auth // .CONTRACT_AUTH // empty')
CONTRACT_DEPOSIT=$(echo "$SERVICE" | jq -r '.contract_deposit // .CONTRACT_DEPOSIT // empty')
CONTRACT_DURATION=$(echo "$SERVICE" | jq -r '.contract_duration // .CONTRACT_DURATION // empty')
CONTRACT_RATE=$(echo "$SERVICE" | jq -r '.contract_rate // .CONTRACT_RATE // empty')
CONTRACT_QPM=$(echo "$SERVICE" | jq -r '.contract_qpm // .CONTRACT_QPM // empty')
CONTRACT_SETTLEMENT=$(echo "$SERVICE" | jq -r '.contract_settlement // .CONTRACT_SETTLEMENT // empty')
CONTRACT_DELEGATE=$(echo "$SERVICE" | jq -r '.contract_delegate // .CONTRACT_DELEGATE // empty')

if [[ -z "$SERVICE_NAME" || -z "$SERVICE_NUM" || -z "$PROVIDER_PUBKEY" || -z "$CLIENT_PUBKEY" ]]; then
echo
echo "Service: $SERVICE_NAME:"
echo "    Service entry is missing required fields."
continue
fi

#
# LIST CONTRACTS
# This pulls a long list of contracts and filters them for the client and provider service.
#

CONTRACTS_JSON="$(arkeod query arkeo list-contracts --output json)"

CONTRACTS=$(echo "$CONTRACTS_JSON" | jq -c \
--arg sn "$SERVICE_NUM" \
--arg prov "$PROVIDER_PUBKEY" \
--arg cli "$CLIENT_PUBKEY" '
[.contract[]
| select(.type == "PAY_AS_YOU_GO")
| select(.service == ($sn | tonumber))
| select(.provider == $prov)
| select(.client == $cli)
| select(.settlement_height == "0")
| select(.deposit != "0")
] | sort_by(.id | tonumber)[]
')

#
# NO CONTRACTS IN SERVICE
# Let's create a contract for the service since it didn't find an active one.
#

if [[ -z "$CONTRACTS" ]]; then

    echo
    echo "Service: $SERVICE_NAME:"
    echo "  No active PAY_AS_YOU_GO contracts found."
    echo "  Attempting to open a new active contract."

    open_arkeo_contract

    continue
fi

#
# CONTRACTS LOOP:
# Process and display information for each active contract
#

PRINTED_SERVICE_HEADER=0

echo "$CONTRACTS" | while read -r CONTRACT; do

    #
    # SERVICE HEADER:
    # Print this header once.
    #

    if [[ "$PRINTED_SERVICE_HEADER" -eq 0 ]]; then
      echo
      echo "Service: $SERVICE_NAME"
      PRINTED_SERVICE_HEADER=1
    fi

    #
    # CONTRACT CALCULATIONS:
    # Various calculations to determine percentages of usage and duration.
    #

    #echo "$CONTRACT" | jq

    CONTRACT_ID=$(echo "$CONTRACT" | jq -r '.id')

    #echo "Contract ID: $CONTRACT_ID"

    DEPOSIT_VAL=$(echo "$CONTRACT" | jq -r '.deposit | tonumber')
    RATE_AMOUNT=$(echo "$CONTRACT" | jq -r '.rate.amount | tonumber')

    CLAIMS_API="$PROVIDER_SENTINEL_API/claims?contract_id=$CONTRACT_ID&client=$CLIENT_PUBKEY"
    NONCE=$(curl -s "$CLAIMS_API" | jq .highestNonce)
    if [ -z "$NONCE" ] || [ "$NONCE" == "null" ]; then
      NONCE=0
    fi

    if [[ -z "$CONTRACT_ID" || -z "$DEPOSIT_VAL" || -z "$NONCE" || -z "$RATE_AMOUNT" ]]; then
      continue
    fi

    #echo "Nonce: $NONCE"
    #echo "Rate Amount: $RATE_AMOUNT"

    USED_AMOUNT=$(($NONCE * $RATE_AMOUNT))
    REMAINDER=$(($DEPOSIT_VAL - $USED_AMOUNT))

    #echo "Deposit Val: $DEPOSIT_VAL"
    #echo "Used Amount: $USED_AMOUNT"
    #echo "Remainder: $REMAINDER"

    if [[ "$DEPOSIT_VAL" -eq 0 ]]; then
      PERCENT_LEFT=0
      PERCENT_USED=0
    else
      PERCENT_LEFT=$(awk "BEGIN {printf \"%.2f\", ($REMAINDER/$DEPOSIT_VAL)*100}")
      PERCENT_USED=$(awk "BEGIN {printf \"%.2f\", 100 - ($REMAINDER/$DEPOSIT_VAL)*100}")
    fi

    CURRENT_HEIGHT=$(arkeod status | jq -r .sync_info.latest_block_height)
    CONTRACT_START=$(echo "$CONTRACT" | jq -r '.height | tonumber')
    CONTRACT_DURATION=$(echo "$CONTRACT" | jq -r '.duration | tonumber')
    CONTRACT_EXPIRE_HEIGHT=$((CONTRACT_START + CONTRACT_DURATION))
    BLOCKS_LEFT=$((CONTRACT_EXPIRE_HEIGHT - CURRENT_HEIGHT))

    if (( CONTRACT_DURATION > 0 )); then
      PERCENT_TIL_EXPIRE=$(awk "BEGIN {printf \"%.2f\", ($BLOCKS_LEFT/$CONTRACT_DURATION)*100}")
      (( $(awk "BEGIN {print ($PERCENT_TIL_EXPIRE < 0)}") )) && PERCENT_TIL_EXPIRE=0
    else
      PERCENT_TIL_EXPIRE=0
    fi

    DURATION_USED=$(awk "BEGIN {printf \"%.2f\", 100 - $PERCENT_TIL_EXPIRE}")


    if (( BLOCKS_LEFT <= 0 )); then

      #
      # CONTRACT SETTLEMENT PHASE
      # The contract has expired, but is still in a settlement phase. The overlap contract should be used for calls.
      #

      echo "  Contract #$CONTRACT_ID, Deposit: ${DEPOSIT_VAL}uarkeo, Expires at Block: $CONTRACT_EXPIRE_HEIGHT"
      echo "    $PERCENT_LEFT% Deposit Remaining"

      # Calculate settlement percentage left
      CONTRACT_START_HEIGHT=$(echo "$CONTRACT" | jq -r '.height | tonumber')
      CONTRACT_DURATION=$(echo "$CONTRACT" | jq -r '.duration | tonumber')
      SETTLEMENT_DURATION=$(echo "$CONTRACT" | jq -r '.settlement_duration | tonumber')
      SETTLEMENT_BLOCKS_LEFT=$(( (CONTRACT_START_HEIGHT + CONTRACT_DURATION + SETTLEMENT_DURATION) - CURRENT_HEIGHT ))
      PERCENT_TIL_SETTLEMENT=$(awk "BEGIN {printf \"%.2f\", ($SETTLEMENT_BLOCKS_LEFT/$SETTLEMENT_DURATION)*100}")
      echo "    $PERCENT_TIL_SETTLEMENT% Settlement Remaining"
    else

      #
      # CONTRACT ACTIVE PHASE
      # Active contract used for calls.
      #

      echo "  Contract #$CONTRACT_ID: Deposit: ${DEPOSIT_VAL}uarkeo, Expires at Block: $CONTRACT_EXPIRE_HEIGHT"
      echo "    $PERCENT_LEFT% Deposit Remaining"
      echo "    $PERCENT_TIL_EXPIRE% Duration Remaining"
    fi

    #
    # CONTRACT OVERLAP CREATION
    # When the active contract reaches a duration threshold, this triggers the creation of an overlapping contract to take over when the active one expires.
    #

    PERCENT_TIL_EXPIRE_INT=${PERCENT_TIL_EXPIRE%.*}
    CONTRACT_COUNT=$(echo "$CONTRACTS" | wc -l)

    if (( PERCENT_TIL_EXPIRE_INT <= CONTRACT_OVERLAP_THRESHOLD )) && (( CONTRACT_COUNT == 1 )); then

      echo
      echo "  Contract ():"
      echo "    Active Contract is getting low."
      echo "    Attempting to open a new contract to use next..."

      open_arkeo_contract

    fi

done

done

echo
```
Notes:
- Ensure the script is run periodically to verify continued connectivity and proper authorization.
- Check logs and outputs regularly to monitor successful execution and identify potential issues early.
- Maintain accurate and up-to-date configurations to reflect your service usage and provider agreements.

## PAYG Contract Test Script

This script identifies the currently active Pay-As-You-Go (PAYG) contract and performs a signed test call to verify connectivity and authorization. It retrieves the latest active contract, computes the next available nonce for claim tracking, generates a cryptographic signature, and makes an authenticated request to the provider's API. This ensures the PAYG contract is functional and ready for use, validating proper setup and ongoing access.

```
#!/bin/bash

# CONFIGURATION:

# CHAIN_ID                  The chain id for Arkeo, as this is needed for signing.
# CLIENT_KEY:               Local key name in your keyring for the client wallet (the party opening/contracts).
# CLIENT_KEYRING:           Keyring backend for the client key ("test", "file", "os", etc.).
# PROVIDER_SERVICE_NAME:    Human-readable name of the blockchain or API service (e.g., "btc-mainnet-fullnode").
# PROVIDER_SERVICE_NUMBER:  Value for the service name, pulled from the enum here: https://raw.githubusercontent.com/arkeonetwork/arkeo/refs/heads/master/common/service.go
# PROVIDER_PUBKEY:          Bech32 public key of the provider (the service operator who receives contract income).
# PROVIDER_SENTINEL_API:    API endpoint for the provider's Sentinel proxy (where client requests are sent for metering/billing).
# PROVIDER_CONTRACT_TYPE:   Type of contract: 0 = subscription (fixed time), 1 = pay-as-you-go (usage-based).

CHAIN_ID="arkeo-main-v2"

CLIENT_KEY="Arkeo-Main-Validator-3"
CLIENT_KEYRING="test"
CLIENT_RAW_PUBKEY=$(arkeod keys show "$CLIENT_KEY" --output json --keyring-backend="$CLIENT_KEYRING" | jq -r .pubkey | jq -r .key)
CLIENT_PUBKEY=$(arkeod debug pubkey-raw "$CLIENT_RAW_PUBKEY" -t secp256k1 | grep 'Bech32 Acc:' | sed "s|Bech32 Acc: ||g")

PROVIDER_SERVICE_NAME="btc-mainnet-fullnode"
PROVIDER_SERVICE_NUMBER=10
PROVIDER_PUBKEY="arkeopub1addwnpepqfn52r6xng2wwfrgz2tm5yvscq42k3yu3ky9cg3kw5s6p0qg7tfx75uwq3z"
PROVIDER_SENTINEL_API="http://127.0.0.1:3636"
PROVIDER_CONTRACT_TYPE=1

# Retrieve the oldest usable contract info (in case there is an overlapping contract).
case "$PROVIDER_CONTRACT_TYPE" in
  0) CONTRACT_TYPE_STRING="SUBSCRIPTION" ;;
  1) CONTRACT_TYPE_STRING="PAY_AS_YOU_GO" ;;
  *) echo "Unknown contract type: $PROVIDER_CONTRACT_TYPE" && exit 1 ;;
esac

CURRENT_HEIGHT=$(arkeod status | jq -r .sync_info.latest_block_height)
# echo "Current Height: $CURRENT_HEIGHT"

CONTRACTS_JSON="$(arkeod query arkeo list-contracts --output json)"
CONTRACT_JSON=$(echo "$CONTRACTS_JSON" | jq -c \
  --arg sn "$PROVIDER_SERVICE_NUMBER" \
  --arg prov "$PROVIDER_PUBKEY" \
  --arg cli "$CLIENT_PUBKEY" \
  --arg type "$CONTRACT_TYPE_STRING" \
  --argjson current_height "$CURRENT_HEIGHT" '
    [.contract[]
      | select(.type == $type)
      | select(.service == ($sn | tonumber))
      | select(.provider == $prov)
      | select(.client == $cli)
      | select(.settlement_height == "0")
      | select(.deposit != "0")
      | select((.height | tonumber) + (.duration | tonumber) > $current_height)
    ] | sort_by(.id | tonumber)[0]'
)

echo
echo "Selected Contract:"
# echo "$CONTRACT_JSON" | jq

if [ -z "$CONTRACT_JSON" ] || [ "$CONTRACT_JSON" == "null" ]; then
  echo "No active PAYG contract found."
  echo
  exit 1
else
  CONTRACT_ID=$(echo "$CONTRACT_JSON" | jq -r '.id')
  echo "  Contract ID: $CONTRACT_ID"
fi

CONTRACT_ID=$(echo "$CONTRACT_JSON" | jq -r '.id')

# GET HIGHEST NONCE USED (Essential for claims, and lowest costs.)
CLAIMS_API="$PROVIDER_SENTINEL_API/claims?contract_id=$CONTRACT_ID&client=$CLIENT_PUBKEY"
HIGHEST_NONCE=$(curl -s "$CLAIMS_API" | jq .highestNonce)
if [ -z "$HIGHEST_NONCE" ] || [ "$HIGHEST_NONCE" == "null" ]; then
  HIGHEST_NONCE=0
fi
NEXT_NONCE=$((HIGHEST_NONCE + 1))

# SIGN THE MESSAGE (Essential for strict/PAYG Contracts)
CHAIN_ID="arkeo-main-v2"
MSG="$CONTRACT_ID:$NEXT_NONCE:$CHAIN_ID"
SIG=$(signhere -u "$CLIENT_KEY" -m "$MSG" | tail -n 1)

# MAKE THE SIGNED REQUEST
ARKAUTH="$CONTRACT_ID:$NEXT_NONCE:$SIG"
echo "  Nonce: $NEXT_NONCE"
echo "  Signature: $SIG"

RESULT=$(curl -s --data-binary "{\"jsonrpc\": \"1.0\", \"id\": \"curltest\", \"method\": \"getblockcount\", \"params\": []}" \
  -H 'content-type: text/plain;' \
  "$PROVIDER_SENTINEL_API/$PROVIDER_SERVICE_NAME?arkauth=$ARKAUTH")

echo "  Result:"
echo "$RESULT" | jq

echo
```

Notes:
- Verify your configuration variables carefully before running this script.
- Regularly test your PAYG contracts to ensure they're valid and operational.
- Maintain sufficient funds and monitor nonce increments closely to avoid disruption in service.