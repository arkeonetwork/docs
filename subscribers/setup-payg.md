# Pay-As-You-Go Contract Setup Guide

Pay-As-You-Go (PAYG) contracts are designed for users who need flexible, usage-based access to provider services on Arkeo. Unlike subscriptions, PAYG contracts allow you to pay only for what you use—ideal for testing, unpredictable workloads, or low-frequency usage where committing to a fixed period isn’t efficient. With PAYG, each individual request or action is explicitly authorized with a digital signature, ensuring you remain in control of every transaction.

Be mindful:
- Each usage must be authorized: You (or your delegate) must sign every usage event, and only authorized claims can be settled and paid out.
- Settlement deadlines matter: If the provider does not settle claims within the contract’s settlement duration, those claims are lost and you won’t be charged for them—but the provider loses payment for those events.
- Deposit and rate limits: Running out of deposit or exceeding configured rate limits will pause your access until you top up or adjust contract terms.
- Nonces for security: Every usage claim must include a unique nonce to prevent replay attacks.

The sections below walk you through creating, using, monitoring, and closing PAYG contracts on Arkeo.

## Create Pay-As-You-Go (PAYG) Contract

To start using a provider’s service on a pay-as-you-go basis, you first need to open a PAYG contract on-chain. This script creates a contract specifying the provider, service, deposit, duration, rate, and any usage limits. Opening the contract is necessary before you can authorize and pay for individual usage events. Always check provider terms (rate, service string, etc.) before running this step to ensure your contract will be accepted.

```
#!/bin/bash

# Variables (update these as needed)
PROVIDER_PUBKEY="<provider-pubkey>"           # Provider's registered public key
PROVIDER_API="<provider-api-host>:26657"      # Provider's arkeod API endpoint
SERVICE="<service-name>"                      # Service string advertised by provider
CONTRACT_TYPE=1                               # 1 = Pay-As-You-Go
CONTRACT_DEPOSIT=<deposit-amount>             # Number of tokens to deposit
CONTRACT_DURATION=<duration-in-blocks>        # Contract duration
CONTRACT_RATE="<rate>"                        # Example: "200uarkeo"
CONTRACT_SETTLEMENT=<settlement-duration>     # Settlement window (blocks)
CONTRACT_QPM=<qpm>                            # Queries per minute limit
CONTRACT_AUTH=0                               # 0 = STRICT, 1 = OPEN
CONTRACT_DELEGATE=""                          # Delegate pubkey (optional)
CLIENT_KEY="<your-client-key>"                # Your client key name (local keyring)
KEYRING_BACKEND="test"                        # Use "file" or "os" for production
FEES="200uarkeo"                              # Transaction fee

# --- Get your client pubkey in Bech32 format ---
CLIENT_RAW_PUBKEY=$(arkeod keys show "$CLIENT_KEY" --output json --keyring-backend="$KEYRING_BACKEND" | jq -r .pubkey | jq -r .key)
CLIENT_PUBKEY=$(arkeod debug pubkey-raw "$CLIENT_RAW_PUBKEY" -t secp256k1 | grep 'Bech32 Acc:' | sed "s|Bech32 Acc: ||g")

echo "Provider pubkey: $PROVIDER_PUBKEY"
echo "Client pubkey:   $CLIENT_PUBKEY"

# --- Open PAYG contract on-chain ---
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
Notes:
- Replace all variables in angle brackets (<...>) with your specific values.
- Make sure your contract terms match the provider’s advertised service and expected rate.
- Use a secure keyring backend (file or os) for production environments, not test.
- Check the result of the transaction to confirm the contract was created successfully.

## Config Pay-As-You-Go (PAYG) Contract

This script allows you to fetch information about your active PAYG contract and then update its configuration—such as whitelist IP addresses, rate limits, or CORS settings- via the provider’s API. For security, every management action must be authorized with a signed message (using your contract and a unique nonce), ensuring that only the contract holder or delegate can make changes.

```
#!/bin/bash

# Set provider API endpoint and contract details
PROVIDER_API="http://127.0.0.1:3636"
CONTRACT_ID=<your-contract-id>
USER_KEY="<your-key-name>"
KEYRING_BACKEND="test"
CLIENT_KEY="<your-key-name>"
CLIENT_PUBKEY=$(arkeod keys show "$CLIENT_KEY" --bech acc --keyring-backend="$KEYRING_BACKEND" --address)

# Sign GET contract info
NONCE_GET=$(date +%s)
MSG_GET="$CONTRACT_ID:$NONCE_GET:"
SIG_GET=$(signhere -u "$USER_KEY" -m "$MSG_GET" | tail -n 1)
ARKAUTH_GET="$CONTRACT_ID:$NONCE_GET:$SIG_GET"
API_URL_GET="$PROVIDER_API/manage/contract/$CONTRACT_ID?arkcontract=$ARKAUTH_GET"

echo "GET Auth: $ARKAUTH_GET"
echo "GET URL: $API_URL_GET"

# Get contract info from the provider
curl -s "$API_URL_GET" | jq .

# Wait briefly to ensure a unique nonce for the next request
sleep 1

# Sign POST request (update config)
NONCE_POST=$(date +%s)
MSG_POST="$CONTRACT_ID:$NONCE_POST:"
SIG_POST=$(signhere -u "$USER_KEY" -m "$MSG_POST" | tail -n 1)
ARKAUTH_POST="$CONTRACT_ID:$NONCE_POST:$SIG_POST"
API_URL_POST="$PROVIDER_API/manage/contract/$CONTRACT_ID?arkcontract=$ARKAUTH_POST"

echo "POST URL: $API_URL_POST"

# Prepare the whitelist update JSON (edit as needed)
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

# Update contract configuration on the provider
curl -X POST -H "Content-Type: application/json" -d "$WHITELIST_JSON" "$API_URL_POST"
echo
```
Notes:
- Replace <your-contract-id> and <your-key-name> with your actual contract ID and local key name.
- Only the contract holder or delegate may update contract configuration.
- Adjust the whitelist or CORS settings as appropriate for your use case.

## View Pay-As-You-Go (PAYG) Contract

To effectively manage your PAYG contract, you should regularly review its current status—such as remaining deposit, rate limits, active claims, and other details. This script securely retrieves contract information from the provider’s API using a signed request. Signing the request with your private key not only authenticates your identity but also ensures the privacy and security of your contract data.

```
#!/bin/bash

# Configuration: Set your provider API, contract ID, key, and keyring backend.
PROVIDER_API="http://127.0.0.1:3636"
CONTRACT_ID=<your-contract-id>
CLIENT_KEY="<your-client-key-name>"
KEYRING_BACKEND="test"

# Fetch your client address (Bech32 format).
CLIENT_PUBKEY=$(arkeod keys show "$CLIENT_KEY" --bech acc --keyring-backend="$KEYRING_BACKEND" --address)
echo "Client Pubkey: $CLIENT_PUBKEY"

# Prepare a unique nonce and message for the GET request.
NONCE_GET=$(date +%s)
MSG_GET="$CONTRACT_ID:$NONCE_GET:"

# Sign the GET message with your client key.
SIG_GET=$(signhere -u "$CLIENT_KEY" -m "$MSG_GET" | tail -n 1)

# Construct the authenticated API URL.
ARKAUTH_GET="$CONTRACT_ID:$NONCE_GET:$SIG_GET"
API_URL_GET="$PROVIDER_API/manage/contract/$CONTRACT_ID?arkcontract=$ARKAUTH_GET"

echo "GET Auth: $ARKAUTH_GET"
echo "GET URL: $API_URL_GET"

# Request contract info from the provider's API and pretty-print it.
curl -s "$API_URL_GET" | jq .
```
Notes:
- Replace <your-contract-id> and <your-client-key-name> with your actual contract ID and client key name.
- Never share your key or signature outputs.
- If running in production, ensure KEYRING_BACKEND is set securely.

## Test Pay-As-You-Go (PAYG) Contract

This script simulates multiple authorized requests against a PAYG contract, making sure each request is correctly signed and counted. Its main purpose is to help subscribers validate rate limits, nonce sequencing, and provider responses under different loads. You’ll use this for integration tests or to ensure that your client and provider are enforcing rate limits and claim settlements as expected.

```
#!/bin/bash

# Usage:
#   ./payg_load_test.sh [RATE_LIMIT] [INTERVAL] [TOTAL_REQUESTS]
#   - RATE_LIMIT:     Max requests per minute (default: 10)
#   - INTERVAL:       Seconds between requests (default: just above the QPM interval)
#   - TOTAL_REQUESTS: Number of requests to make (default: 30)

# --- CONFIGURATION ---
CONTRACT_ID=<your-contract-id>
CHAIN_ID="arkeo-main-v2"
CLIENT_KEY="<your-client-key-name>"
KEYRING_BACKEND="test"
PROVIDER_API="http://127.0.0.1:3636"
SERVICE="<service-name>"
DATA='{"jsonrpc": "1.0", "id": "curltest", "method": "getblockcount", "params": []}'
HEADERS="Content-Type: text/plain"

# Get client Bech32 pubkey
CLIENT_RAW_PUBKEY=$(arkeod keys show "$CLIENT_KEY" --output json --keyring-backend="$KEYRING_BACKEND" | jq -r .pubkey | jq -r .key)
CLIENT_PUBKEY=$(arkeod debug pubkey-raw "$CLIENT_RAW_PUBKEY" -t secp256k1 | grep 'Bech32 Acc:' | sed "s|Bech32 Acc: ||g")

RATE_LIMIT="${1:-10}"
TOTAL_REQUESTS="${3:-30}"

# Calculate request interval (default: 10% slower than max QPM)
if [[ -n "$2" ]]; then
    INTERVAL="$2"
else
    INTERVAL=$(echo "scale=2; 60/$RATE_LIMIT * 1.1" | bc)
fi

echo "Load Test (PAYG): Requests=$TOTAL_REQUESTS, QPM=$RATE_LIMIT, Interval=${INTERVAL}s"
echo "Contract: $CONTRACT_ID"
echo "Client: $CLIENT_KEY"
echo "Endpoint: $PROVIDER_API/$SERVICE"
echo "Data: $DATA"
echo

success=0
rate_limited=0
other=0

start_time=$(date +%s)

# Get starting nonce for the contract
CLAIMS_API="$PROVIDER_API/claims?contract_id=$CONTRACT_ID&client=$CLIENT_PUBKEY"
HIGHEST_NONCE=$(curl -s "$CLAIMS_API" | jq .highestNonce)
if [ -z "$HIGHEST_NONCE" ] || [ "$HIGHEST_NONCE" == "null" ]; then
  HIGHEST_NONCE=0
fi
NEXT_NONCE=$((HIGHEST_NONCE + 1))

for i in $(seq 1 $TOTAL_REQUESTS); do
    loop_start=$(date +%s.%N)
    printf "Request #%d... " "$i"

    # --- SIGN THE MESSAGE FOR THIS NONCE ---
    MSG="$CONTRACT_ID:$NEXT_NONCE:"
    SIG=$(signhere -u "$CLIENT_KEY" -m "$MSG" | tail -n 1)
    ARKAUTH="$CONTRACT_ID:$CLIENT_PUBKEY:$NEXT_NONCE:$SIG"
    ENDPOINT="$PROVIDER_API/$SERVICE?arkauth=$ARKAUTH"

    # --- Make the request ---
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

    NEXT_NONCE=$((NEXT_NONCE + 1))  # Increment for the next request

    # Maintain correct interval timing
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
Notes:
- Replace <your-contract-id>, <your-client-key>, and <service-name> with your actual details.
- Ensure you have the appropriate permissions, deposit, and contract status before testing.
- This script will help you confirm rate limiting and signature-based PAYG access is working as intended.

## Close Pay-As-You-Go (PAYG) Contract

Closing a PAYG contract formally ends your usage agreement with a provider on the Arkeo network. This is needed when you no longer wish to authorize further usage or want to trigger final settlement of outstanding claims. Once a PAYG contract is closed, you cannot authorize new usage, and the provider can settle any remaining claims for payment. Always close your contract when finished to ensure all activity is finalized and the contract is removed from your active list.

```
#!/bin/bash

# Define your API endpoint, contract ID, client key, and keyring backend.
PROVIDER_API="127.0.0.1:26657"
CONTRACT_ID=<your-contract-id>
CLIENT_KEY="<your-client-key-name>"
KEYRING_BACKEND="test"
FEES="200uarkeo"

# Retrieve your client Bech32 public key for contract closure.
CLIENT_RAW_PUBKEY=$(arkeod keys show "$CLIENT_KEY" --output json --keyring-backend="$KEYRING_BACKEND" | jq -r .pubkey | jq -r .key)
CLIENT_PUBKEY=$(arkeod debug pubkey-raw "$CLIENT_RAW_PUBKEY" -t secp256k1 | grep 'Bech32 Acc:' | sed "s|Bech32 Acc: ||g")

echo "Client pubkey: $CLIENT_PUBKEY"

# Submit the transaction to close the PAYG contract.
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
- Set KEYRING_BACKEND and PROVIDER_API as appropriate for your environment.
- After closing, verify contract status with arkeod query arkeo show-contract <id>.