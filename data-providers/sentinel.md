Sentinel
========
Sentinel is a custom built reverse proxy (similar to nginx). Its role is be
the front facing services that proxies requests to backend services (ie
an RPC). It is also in charge of authentication/authorization as well as rate
limiting services.

## API Documentation
### Metadata
To retrieve the metadata of any given data provider query the following
endpoint

```bash
$ curl -s http://<ip-address>:3636/metadata.json | jq
{
  "config": {
    "moniker": "n/a",
    "website": "n/a",
    "description": "n/a",
    "location": "n/a",
    "port": "3636",
    "source_chain": "http://arkeo:1317",
    "event_stream_host": "arkeo:26657",
    "claim_store_location": "${HOME}/.arkeo/claims",
    "provider_pubkey": "arkeopub1-----------------",
    "free_tier_rate_limit": 10,
  },
  "version": "0.1.0"
}
```

### Active Contracts
This endpoint returns a list of active contracts for this specific data
provider

```bash
$ curl -s http://<ip-address>:3636/active-contract/<service>/<spender_pubkey> | jq
```

### Claims
This endpoint returns the latest claim for a given contract Id.

```bash
$ curl -s http://<ip-address>:3636/claim/<contract_id> | jq
```

The open claims endpoint is used to get a list of claims to contracts that
haven't been claimed yet. Anything showing up in this endpoint is available to
be claimed on chain.

```bash
$ curl -s http://<ip-address>:3636/open-claims | jq
```

## Authentication
To make an authenticated request, one must first have an open contract with a
given data provider. Once you have an open contract with a contract Id, add
query arg do your request as such

```bash
/?arkauth=<contract_id>:<nonce>:<signature>
```

The `nonce` refers to counter that increments on each request to the data
provider. If the client doesn't know the current `nonce`, query the `claim`
(`/claim/<contract_id>`) which will include both the nonce and the signature
to prove its correct.

If your contract is an "open" authorization, you can just supply the
`contract_id` (no need to `nonce` or `signature`).

To create the signature, use your private key to sign the following text

```bash
<contract_id>:<nonce>
```

The signature produce should be hex encoded into a string for transmission
([sample](https://pkg.go.dev/encoding/hex#EncodeToString)]. There is a command
line tool to creating this signature in the arkeo codebase, called
[signhere](https://github.com/arkeonetwork/arkeo/tree/master/tools/signhere)
