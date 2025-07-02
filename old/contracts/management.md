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

## Contract Configuration
After a contract is open, there are some attributes that are configurable by the contract owner. Attributes such as IP white listing, cross-origin resource sharing (CORS), and per user rate limiting.

### Authentication
In order to read or write a contract configuration, the contract owner (with
their private key) needs to authenticate with the data provider. This is done
by making a `GET` or `POST` request with a signature. That request will look
something like

```
GET http://<data provider ip
address>:3636/manage/contract/<id>?arkcontract=<id>:<nonce>:<sig>
```

The `<id>` is the contract id, `<nonce>` is an integer that is larger than the
last one. While this number is somewhat arbitrary, the recommendation is to
use the current [epoch](https://www.epoch101.com/). If you need more
granularity than per second, use milliseconds. And the `<sig>` is the
signature of `<id>:<nonce>`. The signature produce should be hex encoded into
a string for transmission
([sample](https://pkg.go.dev/encoding/hex#EncodeToString)]. There is also a
command line tool to creating this signature in the arkeo codebase, called
[signhere](https://github.com/arkeonetwork/arkeo/tree/master/tools/signhere)

### Configuration
When making a `GET` request, you will get back some JSON structured like so.

```
{
    "contract_id": 1234,
    "last_timestamp": 1684884769,
    "per_user_rate_limit": 10, // NOTE: represented in per minute basis
    "white_listed_ip_address": [
        "192.168.0.1",
        "8.8.8.8"
    ],
    "cors": {
        "allow_origins": ["*"],
        "allow_methods": ["GET", "POST"],
        "allow_headers": ["header1", "header2"]
    }
}
```

Once you have the current state of the contract configuration. You can
alter/change any aspect to the json and `POST` it back to the data provider

## Close a Contract

If the contract is a subscription, it can be cancelled. Pay-as-you-go isn’t available to cancel as you can stop making requests as a form of cancelling (providers can cancel though). Closing a contract should also trigger a payout to the provider.

```bash
$ arkeod tx arkeo close-contract -y --from <user> --keyring-backend file --node "tcp://seed.arkeo.network:26657" -- arkeopub1addwnpepqtrc0rrpkwn2esula68zl3dvqqfxfjhr5dyfxy3uq97dssntrq8twhy9nvu btc-mainnet-fullnode "<your pubkey>"
```
