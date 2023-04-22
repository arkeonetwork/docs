Contract Management
===================

## Open a Contract

Open both a subscription and pay-as-you-go contract (not at the same time, as only one contract is allowed open at a time between a provider and client/delegate) with the sole data provider.

```bash
$ PROVIDER="tarkeo..." # the provider's pubkey
$ SERVICE="btc-mainnet-fullnode"
$ CTYPE=0 # contract type, 0 is subscription, 1 is pay-as-you-go
$ DEPOSIT=<amt> # amount of tokens you want to deposit. Subscriptions should make sense in that duration and rate equal deposit
$ DURATION=<blocks> # number of blocks to make a subscription. There are lower and higher limits to this number
$ RATE=<rate> # should equal the porvider's rate which you can lookup at (`curl http://seed.arkeo.network:3636/metadata.json | jq .`)
$ SETTLEMENT_DURATION=<duration> # this number should equal the same number
$ QPM=<queries per minute> # this set the rate limit for the contract. A
higher value will come with a higher cost
$ AUTH=0 # defines if the contract has strict authorization (0)  or open (1)
$ DELEGATE="" # if you'd like to have a different private key spend from the
contract, but that pubkey here

that the data provider has configured for themselves
$ arkeod tx arkeo open-contract -y --from <user> --keyring-backend file --node "tcp://seed.arkeo.network:26657" -- "$PROVIDER" "$SERVICE" "<your pubkey>" "$CTYPE" "$DEPOSIT" "$DURATION" $RATE $QPM $SETTLEMENT_DURATION $AUTH $DELEGATE
```

## Close a Contract

If the contract is a subscription, it can be cancelled. Pay-as-you-go isn’t available to cancel as you can stop making requests as a form of cancelling (providers can cancel though). Closing a contract should also trigger a payout to the provider.

```bash
$ arkeod tx arkeo close-contract -y --from <user> --keyring-backend file --node "tcp://seed.arkeo.network:26657" -- arkeopub1addwnpepqtrc0rrpkwn2esula68zl3dvqqfxfjhr5dyfxy3uq97dssntrq8twhy9nvu btc-mainnet-fullnode "<your pubkey>"
```
