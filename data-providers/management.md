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
arkeod tx arkeo bond-provider -y --from "$USER" --keyring-backend file -- "$PUBKEY" "$SERVICE" "$BOND"
```

You must do this per service you intend to run (ie Bitcoin, Ethereum, Gaia,
etc).

## Modifying
Once you have registered your data provider and services, you'll want to
configure additional information.

| Attribute Name         | Attribute Type | Example                          | Notes                                                                                                   |
| ---------------------- | -------------- | -------------------------------- | ------------------------------------------------------------------------------------------------------- |
| Service                | string         | btc-mainnet-fullnode            |                                                                                                         |
| MetadataUri            | string         | http://<my ip>:3636/metadata.json | This should be a fully qualified URI                                                                    |
| MetadataNonce          | uint64         | 45                               | This should increment each time you modify or change the contents of metadata.json within your sentinel |
| Status                 | enum           | Online                           | This allows you to signal that you're in a maintenance window and no new contracts would be open during this time.          |
| MinContractDuration    | int64          | 5                                | This sets a minimum contract duration                                                                  |
| MaxContractDuration    | int64          | 432000                           | This sets a maximum contract duration. This ensures that contracts aren't open for too long, making it hard to adjust pricing. |
| SubscriptionRate       | []types.Coin   | "40uarkeo,..."                     | This allows you to set your prices for subscription contracts. You can specify any IBC-enabled assets like ATOM or ARKEO or USDC. |
| PayAsYouGo             | []types.Coin   | "90uarkeo,..."                     | Same as above but for pay-as-you-go contracts.                                                         |
| SettlemenDuration      | int64          | 1000                             | This gives the data provider additional time after a contract has expired to make any last-minute claims for rewards (for pay-as-you-go contracts). |


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
