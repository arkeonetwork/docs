Data Provider Management
========================

## Registering
A data provider can be registered or de-registered by bonding or unbonding.
There is a min bond size required for a data provider in order for it to be
eligible for users and dapps to open contracts with, but at the time of this
writing, that amount is `1 arkeo`. You are allowed to unbond while a data
provider has open contracts.

To bond or unbond, broadcast a `MsgBondProvider` transaction.
```
Provider    []byte
Service     enum
Bond        string
```

The bond amount can be positive or negative (positive to bond, negative to
unbond).

To do this via the `cli`,
```bash
USER=wallet
SERVICE=eth-mainnet-fullnode
BOND=1000000
RAWPUBKEY=$(arkeod keys show "$USER" -p | jq -r .key)
PUBKEY=$(arkeod debug pubkey-raw $RAWPUBKEY | grep 'Bech32 Acc:' | sed "s|Bech32 Acc: ||g")
arkeod tx arkeo bond-provider -y --from "$USER" --fees 50uarkeo --keyring-backend file -- "$PUBKEY" "$SERVICE" "$BOND"
```

You must do this per service you intend to run (ie Bitcoin, Ethereum, Gaia,
etc).

## Modifying
Once you have registered your data provider and services, you'll want to
configure additional information.

| **Attribute Name**      | **Attribute Type** | **Example**                              | **Notes**                                                                                                                                      |
|--------------------------|--------------------|------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------|
| **Service**             | `string`          | `btc-mainnet-fullnode`                  | The name of the service being provided.                                                                                                      |
| **MetadataUri**         | `string`          | `http://{SENTINEL_PUBLIC_IP_OR_DOMAIN}:3636/metadata.json` | Must be a fully qualified URI pointing to the metadata file.                                                                                   |
| **MetadataNonce**       | `uint64`          | `45`                                    | Increments with each modification to `metadata.json`.                                                                                         |
| **Status**              | `enum`            | `Online`                                | Signals service status. For example, during maintenance, no new contracts can be opened.                                                     |
| **MinContractDuration** | `int64`           | `5`                                     | Specifies the minimum duration of contracts in seconds.                                                                                       |
| **MaxContractDuration** | `int64`           | `432000`                                | Specifies the maximum duration of contracts in seconds. Prevents long contracts that hinder pricing adjustments.                              |
| **SubscriptionRate**    | `[]types.Coin`    | `"40uarkeo,..."`                        | Defines the price for subscription contracts. Accepts IBC-enabled assets like ARKEO, ATOM, or USDC.                                           |
| **PayAsYouGo**          | `[]types.Coin`    | `"90uarkeo,..."`                        | Defines the price for pay-as-you-go contracts. Same format as `SubscriptionRate`.                                                             |
| **SettlemenDuration**   | `int64`           | `1000`                                  | The time (in seconds) allowed for the provider to make claims after a contract expires (relevant for pay-as-you-go contracts).                |


To do this via the `cli`,
```bash
arkeod tx arkeo mod-provider -y --from "$USER" --keyring-backend file -- "$PUBKEY" "$SERVICE" "$METADATAURI" $STATUS $MIN_CONTRACT_DURATION $MAX_CONTRACT_DURATION $SUBSCRIPTION_RATE  $PAY_AS_YOU_GO_RATE $SETTLEMENT_DURATION
```

## Claim Rewards for a Provider
For subscription contract, one does not need to claim the rewards, as they are
granted automatically when the contract expires. But someone can claim the
rewards earned so far (prorata) at any given time (if desired). For
pay-as-you-go contract, the data provider MUST claim the rewards in order to
receive them. If the provider doesn't do this, then the income they've earned
will not be realized when the contract expires.

Any private key can claim the rewards on the data provider's behalf, it does
not have to be the data provider's private key making the claim.

To make a claim, you need to acquire the signature of the contract owner
created when making the latest request. This can be done via querying the data
provider's sentinel.
```bash
$ curl -s http://<ip-address>:3636/open-claims | jq
```
This endpoint will give you the contract id, nonce, and signature. Broadcast a
transaction with those values as shown below.

```bash
$ ID=<id> # the contract ID
$ NONCE=<num> # the nonce represents the number of queries made between the client/provider and provider during this contract
$ SIGNATURE="<signature>" 
$ arkeod tx arkeo claim-contract-income -y --from <user> --keyring-backend file --node "tcp://seed.arkeo.network:26657" -- "$ID" "$NONCE" "$SIGNATURE"
```

**NOTE** If you are claiming income for an open subscription contract, no signature is
required.
