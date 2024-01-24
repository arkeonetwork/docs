Contract Query
===================

## Query a Contract
To query a contract you must made a request to the reverse proxy (sentinel) of
the data provider. You must include in the path the name of the service you'd
like to query and supply authentication information (if required)

### Making an free tier query

To make a free tier query, just query the service directly (not contract or
authorization information is required).

An example of this would be
```
curl -vvv --data-binary '{"jsonrpc": "1.0", "id": "curltest", "method": "ping", "params": []}' -H 'content-type: text/plain;' http://seed.arkeo.network:3636/btc-mainnet-fullnode
```

### Making an open contract query
Open contracts are contracts that are paid for, but allows anyone to query
that contract (without authentication or authorization). This query looks
almost the same as a free tier query, but specifies the contract id number as
a query arg

An example of this would be
```
curl -vvv --data-binary '{"jsonrpc": "1.0", "id": "curltest", "method": "ping", "params": []}' -H 'content-type: text/plain;' http://seed.arkeo.network:3636/btc-mainnet-fullnode?arkauth=<contract_id>
```

### Making an authenticated contract query
To make an authenticated request, one must provide the contract id, nonce, and
signature.

```
curl -vvv --data-binary '{"jsonrpc": "1.0", "id": "curltest", "method": "ping", "params": []}' -H 'content-type: text/plain;' http://seed.arkeo.network:3636/btc-mainnet-fullnode?arkauth=<contract_id>:<nonce>:<signature>
```

The `nonce` refers to counter that increments on each request to the data
provider. If the client doesn't know the current `nonce`, query the `claim`
(`/claim/<contract_id>`) which will include both the nonce and the signature
to prove its correct.

To create the signature, use your private key to sign the following text

```bash
<contract_id>:<nonce>
```

The signature produce should be hex encoded into a string for transmission
([sample](https://pkg.go.dev/encoding/hex#EncodeToString)). There is a command
line tool to creating this signature in the arkeo codebase, called
[signhere](https://github.com/arkeonetwork/arkeo/tree/master/tools/signhere)

In addition, there is a cli tool called
[curleo](https://github.com/arkeonetwork/arkeo/tree/master/tools/curleo),
which makes this easier to do, as url forming, nonce, and signing is all
handled for you.
