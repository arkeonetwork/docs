# Subscription Contract Setup Guide

Subscription contracts on Arkeo provide clients with predictable, recurring access to provider services for a set period and rate. This model is ideal when you require continuous service without having to sign every usage event. As a subscriber, you pay upfront for a specified duration, during which you’re entitled to the provider’s resources within the agreed query-per-minute (QPM) and rate limits.

### Be mindful:
- **Subscription contracts are prepaid:** Your deposit covers the entire contract duration and rate; unused time or quota is not refundable if the contract is closed early.
- **Rate limits apply:** Exceeding the allowed QPM may result in throttling or denied service for the remainder of the interval.
- **Contract duration is fixed:** Extensions or changes require opening a new contract.
- **Closure triggers settlement:** Closing a contract early will immediately settle usage and may forfeit any remaining value.

The following sections walk you through opening, configuring, monitoring, testing, and closing a subscription contract on Arkeo.

## Opening a Subscription Contract

Before you can use a provider’s resources on Arkeo, you need to open a subscription contract. This script prepares all required contract parameters—including the provider’s public key, the chosen service, deposit, contract duration, rate, and authorization options—and submits a transaction to open the contract on-chain. Running this script sets up your access and guarantees your service level for the defined duration, provided you stay within the contract’s rate and query-per-minute limits.

```
#!/bin/bash

# Configuration: Set these values appropriately
PROVIDER_API="127.0.0.1:26657"                   # RPC endpoint for submitting tx
PROVIDER_PUBKEY="<provider-bech32-pubkey>"       # Provider's registered pubkey
SERVICE="<service-name>"                         # Service string as advertised by provider
CONTRACT_TYPE=0                                  # 0 = Subscription, 1 = Pay-As-You-Go
CONTRACT_DEPOSIT=172800000                       # Deposit (tokens); for subscriptions: rate * duration
CONTRACT_DURATION=86400                          # Duration (in blocks)
CONTRACT_RATE="200uarkeo"                        # Must match provider's published rate
CONTRACT_SETTLEMENT=1000                         # Settlement window (blocks)
CONTRACT_QPM=10                                  # Queries per minute (QPM)
CONTRACT_AUTH=1                                  # 0 = STRICT, 1 = OPEN
CONTRACT_DELEGATE=""                             # (Optional) Delegate pubkey
CLIENT_KEY="<your-client-key>"                   # Your local key name in arkeod keyring
KEYRING_BACKEND="test"                           # Or \"os\" / \"file\" for better security
FEES="200uarkeo"                                 # Transaction fee

# --- Get client pubkey in bech32 format ---
CLIENT_RAW_PUBKEY=$(arkeod keys show "$CLIENT_KEY" --output json --keyring-backend="$KEYRING_BACKEND" | jq -r .pubkey | jq -r .key)
CLIENT_PUBKEY=$(arkeod debug pubkey-raw "$CLIENT_RAW_PUBKEY" -t secp256k1 | grep 'Bech32 Acc:' | sed "s|Bech32 Acc: ||g")

echo "Provider pubkey: $PROVIDER_PUBKEY"
echo "Client pubkey:   $CLIENT_PUBKEY"

# --- Submit transaction to open the contract ---
arkeod tx arkeo open-contract \
"$PROVIDER_PUBKEY" \
"$SERVICE" \
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
--fees="$FEES" \
--keyring-backend="$KEYRING_BACKEND" \
--node tcp://$PROVIDER_API \
-y
```
Note:
- Replace all placeholder values (inside <...>) with real details for your use case.
- Always check the provider’s advertised service name and rates before creating a contract.
- Make sure your keyring contains the correct client key and has enough balance for the deposit and fees.

## Configuring a Subscription Contract

After you have opened a subscription contract, you may want to fetch its details or update access configurations, such as IP whitelists or rate limits, using the provider’s Sentinel API. The script below securely signs API requests with your client key, retrieves contract information, and then updates the contract configuration. This is useful for enforcing access controls and customizing allowed usage from your end.

```
#!/bin/bash

# Configuration
PROVIDER_API="http://127.0.0.1:3636"
CONTRACT_ID=1
USER_KEY="your-client-key"
KEYRING_BACKEND="test"
CLIENT_KEY="your-client-key"
CLIENT_PUBKEY=$(arkeod keys show "$CLIENT_KEY" --bech acc --keyring-backend="$KEYRING_BACKEND" --address)

# --- Sign and GET contract info ---
NONCE_GET=$(date +%s)
MSG_GET="$CONTRACT_ID:$NONCE_GET:"
SIG_GET=$(signhere -u "$USER_KEY" -m "$MSG_GET" | tail -n 1)
ARKAUTH_GET="$CONTRACT_ID:$NONCE_GET:$SIG_GET"
API_URL_GET="$PROVIDER_API/manage/contract/$CONTRACT_ID?arkcontract=$ARKAUTH_GET"

# Fetch contract details
curl -s "$API_URL_GET" | jq .

# --- Wait to ensure a unique POST nonce ---
sleep 1

# --- Sign POST request to update config ---
NONCE_POST=$(date +%s)
MSG_POST="$CONTRACT_ID:$NONCE_POST:"
SIG_POST=$(signhere -u "$USER_KEY" -m "$MSG_POST" | tail -n 1)
ARKAUTH_POST="$CONTRACT_ID:$NONCE_POST:$SIG_POST"
API_URL_POST="$PROVIDER_API/manage/contract/$CONTRACT_ID?arkcontract=$ARKAUTH_POST"

# Prepare whitelist and config JSON (edit as needed)
read -r -d '' WHITELIST_JSON <<EOF
{
  "white_listed_ip_addresses": [
    "127.0.0.1",
    "192.168.0.1"
  ],
  "per_user_rate_limit": 1000,
  "cors": {
    "allow_origins": ["*"],
    "allow_methods": ["GET", "POST", "PUT", "DELETE", "OPTIONS"],
    "allow_headers": ["*"]
  }
}
EOF

# --- POST updated contract configuration ---
curl -X POST -H "Content-Type: application/json" -d "$WHITELIST_JSON" "$API_URL_POST"
echo
```
Notes:
- Replace placeholder values (like <provider-host>, <your-contract-id>, <your-client-key-name>) with your actual details.
- The first half of the script securely fetches contract info; the second half (optional) shows how to update settings if needed.
- Never commit your private keys or sensitive environment variables to source control.
- Only authorized users (contract owner or delegate) can view or change the contract’s config.

## View Subscription Contract

To effectively manage your subscription contract, you may need to view its current status, usage, and configuration as stored by the provider. This script securely fetches up-to-date information about your contract, using a signed request to authenticate your access. Running this ensures you can monitor your claims, see allowed limits, and troubleshoot or audit your subscription as needed.

```
#!/bin/bash

# Configuration (replace values with your actual details)
PROVIDER_API="http://127.0.0.1:3636"       # Provider's Sentinel API endpoint
CONTRACT_ID=1                              # Your contract ID
USER_KEY="your-client-key"                 # Your local key name
KEYRING_BACKEND="test"                     # Keyring backend (use 'file' or 'os' for production)

# Fetch your public address
CLIENT_PUBKEY=$(arkeod keys show "$USER_KEY" --bech acc --keyring-backend="$KEYRING_BACKEND" --address)
echo "Client Pubkey: $CLIENT_PUBKEY"

# Display claims endpoint
CLAIMS_API="$PROVIDER_API/claims?contract_id=$CONTRACT_ID&client=$CLIENT_PUBKEY"
echo "Claims API: $CLAIMS_API"

# Prepare signed nonce for authenticated GET request
NONCE_GET=$(date +%s)
MSG_GET="$CONTRACT_ID:$NONCE_GET:"
SIG_GET=$(signhere -u "$USER_KEY" -m "$MSG_GET" | tail -n 1)
ARKAUTH_GET="$CONTRACT_ID:$NONCE_GET:$SIG_GET"
API_URL_GET="$PROVIDER_API/manage/contract/$CONTRACT_ID?arkcontract=$ARKAUTH_GET"

echo "GET Auth: $ARKAUTH_GET"
echo "GET URL: $API_URL_GET"

# Retrieve contract info from the provider's API
curl -s "$API_URL_GET" | jq .
```
Note:
- Replace your-client-key-name and CONTRACT_ID with your actual values.
- Only use keyring-backend test for local/testing environments; use a secure backend in production.

## Test Subscription Contract

Before running this script, ensure your subscription contract is active and the provider’s API endpoint is accessible.

This script is designed to simulate usage of your subscription contract by sending repeated requests to the provider’s API endpoint. It helps you verify whether the contract’s rate limits (queries per minute) are enforced and lets you observe the provider’s responses to typical usage patterns. This is useful for troubleshooting, auditing, and making sure you do not get unexpectedly throttled during actual use.

```
#!/bin/bash

# Usage:
#   ./test_subscription.sh [RATE_LIMIT] [INTERVAL] [TOTAL_REQUESTS]
#   RATE_LIMIT:     max requests per minute (default: 10)
#   INTERVAL:       seconds between requests (default: calculated to slightly exceed limit)
#   TOTAL_REQUESTS: number of requests to make (default: 30)

CONTRACT_ID=1
ENDPOINT="http://127.0.0.1:3636/btc-mainnet-fullnode?arkauth=$CONTRACT_ID"
DATA='{"jsonrpc": "1.0", "id": "curltest", "method": "getblockcount", "params": []}'
HEADERS="Content-Type: text/plain"

RATE_LIMIT="${1:-10}"
TOTAL_REQUESTS="${3:-30}"

# Calculate interval: if not specified, use 10% slower than max QPM to avoid 429 errors
if [[ -n "$2" ]]; then
    INTERVAL="$2"
else
    INTERVAL=$(echo "scale=2; 60/$RATE_LIMIT * 1.1" | bc)
fi

echo "Load Test (Subscription): Requests=$TOTAL_REQUESTS, QPM=$RATE_LIMIT, Interval=${INTERVAL}s"
echo "Contract: $CONTRACT_ID"
echo "Endpoint: $ENDPOINT"
echo "Data: $DATA"
echo

success=0
rate_limited=0
other=0

start_time=$(date +%s)

for i in $(seq 1 $TOTAL_REQUESTS); do
    loop_start=$(date +%s.%N)
    printf "Request #%d... " "$i"

    response=$(curl -s -w "\n%{http_code}" -X POST --data-binary "$DATA" -H "$HEADERS" "$ENDPOINT")
    http_code=$(echo "$response" | tail -n1)
    http_body=$(echo "$response" | sed '$d')

    if [ "$http_code" == "200" ]; then
        echo "✅ 200 OK"
        ((success++))
    elif [ "$http_code" == "403" ] || [ "$http_code" == "429" ]; then
        echo "🚫 Rate Limited ($http_code)"
        ((rate_limited++))
    else
        echo "❓ $http_code"
        ((other++))
    fi

    loop_end=$(date +%s.%N)
    elapsed=$(echo "$loop_end - $loop_start" | bc)

    sleep_time=$(echo "$INTERVAL - $elapsed" | bc)
    if (( $(echo "$sleep_time > 0" | bc -l) )); then
        sleep $sleep_time
    fi
done

end_time=$(date +%s)
duration=$((end_time - start_time))
if [ "$duration" -gt 0 ]; then
    qpm=$((success * 60 / duration))
else
    qpm=0
fi

echo
echo "Total Requests:    $TOTAL_REQUESTS"
echo "Duration:          $duration seconds"
echo "Successful (200):  $success"
echo "Rate Limited:      $rate_limited"
echo "Other Errors:      $other"
echo "Approx. QPM:       $qpm"
```
- Adjust CONTRACT_ID and ENDPOINT to match your actual provider and contract.
- The script prints a summary at the end to help you understand your contract’s rate-limiting behavior.

## Close Subscription Contract

Closing a subscription contract is the process of formally ending your agreement with the provider on the Arkeo blockchain. This is important when you no longer need the service or want to finalize and settle usage up to the current point. Closing the contract will stop further access, trigger settlement (so the provider can get paid), and, in some cases, release any refundable deposit. Always ensure you’re ready to stop the service, as this action is final for the contract period.

```
#!/bin/bash

# Set variables for the provider API endpoint, contract ID, client key, and keyring backend.
PROVIDER_API="127.0.0.1:26657"
CONTRACT_ID=<your-contract-id>
CLIENT_KEY="<your-client-key-name>"
KEYRING_BACKEND="test"
FEES="200uarkeo"

# Fetch your client Bech32 public key for contract closure.
CLIENT_RAW_PUBKEY=$(arkeod keys show "$CLIENT_KEY" --output json --keyring-backend="$KEYRING_BACKEND" | jq -r .pubkey | jq -r .key)
CLIENT_PUBKEY=$(arkeod debug pubkey-raw "$CLIENT_RAW_PUBKEY" -t secp256k1 | grep 'Bech32 Acc:' | sed "s|Bech32 Acc: ||g")

echo "Client pubkey: $CLIENT_PUBKEY"

# Submit the transaction to close the subscription contract.
arkeod tx arkeo close-contract \
"$CONTRACT_ID" \
"$CLIENT_PUBKEY" \
--from="$CLIENT_KEY" \
--fees="$FEES" \
--keyring-backend="$KEYRING_BACKEND" \
--node tcp://$PROVIDER_API \
-y
```

Notes:
- Replace <your-contract-id> and <your-client-key-name> with your actual contract ID and local key name.
- Ensure your keyring backend and node address match your setup (for production, do not use "test" for keyring backend).
- After closing, you can verify contract status on-chain using arkeod query arkeo show-contract <id>.
