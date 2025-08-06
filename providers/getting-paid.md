# Getting Paid as a Provider

As an Arkeo provider, ensuring timely settlements of your earned income is crucial. This guide outlines how to settle your claims, explains when payments occur, and highlights important considerations to ensure you receive payments without interruption.

## When Providers Get Paid

- Providers get paid at the end of each contract period.
- Payments are consistent across both Subscription and Pay-As-You-Go (PAYG) contracts.
- Providers must explicitly submit claims periodically throughout the contract's settlement duration.

> Important: If you fail to settle your claim income before the contract's settlement duration expires, you will permanently lose those funds.

## How to Settle Your Claims

### What this function does:

- **Set variables:** Defines important values like your provider key, keyring backend, API endpoint, and blockchain fees.
- **Retrieve provider account:** Fetches your blockchain account address.
- **Get open claims:** Queries your provider API for claims that are ready for settlement. 
- **Process claims:** For each open claim:
  - Extracts the contract ID, nonce, and signature. 
  - Queries your blockchain account to determine the correct transaction sequence. 
  - Submits the claim to the blockchain using arkeod tx arkeo claim-contract-income, specifying the necessary parameters. 
- **Sleep interval:** Adds a short delay between transactions to ensure stability.

Replace the placeholder values (like your-provider-key) with your actual details.

```
#!/bin/bash
set -e

PROVIDER_KEY="your-provider-key"
KEYRING_BACKEND="test"
PROVIDER_API="http://127.0.0.1:3636"
CHAIN_ID="arkeo-main-v1"
FEES="200uarkeo"

PROVIDER_ACCOUNT=$(arkeod keys show "$PROVIDER_KEY" --bech acc --keyring-backend "$KEYRING_BACKEND" --address)

OPEN_CLAIMS=$(curl -sL "$PROVIDER_API/open-claims" | jq -c '.[] | select(.claimed == false)')

if [[ -z "$OPEN_CLAIMS" ]]; then
echo "No open claims to process."
exit 0
fi

echo "$OPEN_CLAIMS" | while read -r CLAIM; do
contract_id=$(echo "$CLAIM" | jq -r '.contract_id')
nonce=$(echo "$CLAIM" | jq -r '.nonce')
signature=$(echo "$CLAIM" | jq -r '.signature')

    RAW_ACCOUNT=$(arkeod query auth account "$PROVIDER_ACCOUNT" --output json)
    CURRENT_SEQ=$(echo "$RAW_ACCOUNT" | jq -r '.account.sequence // .account.value.sequence // .account.base_account.sequence // .account.base_account.value.sequence' 2>/dev/null)

    if [ -z "$CURRENT_SEQ" ]; then
      echo "ERROR: Failed to get current sequence."
      exit 1
    fi

    echo "Submitting claim for contract $contract_id, nonce $nonce."
    arkeod tx arkeo claim-contract-income "$contract_id" "$nonce" "$signature" nil \
      --from "$PROVIDER_KEY" \
      --keyring-backend "$KEYRING_BACKEND" \
      -b sync \
      --sequence "$CURRENT_SEQ" -y

    sleep 2
    echo "Claim submitted."
done
```

## Recommended Claim Submission Frequency

- Run this following script on a cron to periodically submit claims for Arkeo blockchain settlement.
- This script should be run regularly (e.g., hourly or daily), ensuring claims are always settled within your contract's defined settlement duration.
- Frequent submissions help reduce the risk of losing income due to missed settlements.

## Security and Operational Best Practices

- Use a dedicated hot wallet (keyring-backend test) specifically for automated scripts, funded minimally.
- Regularly monitor claim submissions and your provider account balance.
- Keep logs and review settlement script outputs periodically.

## Troubleshooting and Monitoring

Check recent claim statuses and account balances:
```
arkeod query bank balances <your-provider-account-address>
```
Review settlement logs regularly to detect and address issues promptly.