# Provider Management

This document outlines management tasks for maintaining your Arkeo provider services, including querying your provider details and understanding the service enum mappings used in Arkeo.

## Querying Provider Details

To view detailed information about your registered provider, filter by your provider public key (pub_key) using the following command:

```
arkeod query arkeo list-providers --output json | jq '.provider[] | select(.pub_key=="<your-provider-pubkey>")'
```

## Listing Active Contracts by Provider

To list all active contracts associated with your provider public key (contracts where settlement height is 0), use the following command:

```
arkeod query arkeo list-contracts --output json | jq '.contract[] | select(.provider=="<your-provider-pubkey>" and .settlement_height=="0")'
```

## Best Practices

- Regularly query your provider details to ensure your service information is current and accurate.
- Periodically check your active contracts to manage settlements effectively.
- Understand the service mapping clearly when adding or updating your provider offerings.
- Regular monitoring and periodic verification ensure maximum reliability and client satisfaction.

## Arkeo Service Enum Mapping

Below is a comprehensive reference mapping between service names and their numeric identifiers, as used within Arkeo:

| Service Name                     | Service Number |
|----------------------------------|----------------|
| unknown                          | 0              |
| mock                             | 1              |
| arkeo-mainnet-fullnode           | 2              |
| avax-mainnet-fullnode            | 3              |
| avax-mainnet-archivenode         | 4              |
| bch-mainnet-fullnode             | 5              |
| bch-mainnet-lightnode            | 6              |
| bnb-mainnet-fullnode             | 7              |
| bsc-mainnet-fullnode             | 8              |
| bsc-mainnet-archivenode          | 9              |
| btc-mainnet-fullnode             | 10             |
| btc-mainnet-lightnode            | 11             |
| cardano-mainnet-relaynode        | 12             |
| gaia-mainnet-rpc                 | 13             |
| doge-mainnet-fullnode            | 14             |
| doge-mainnet-lightnode           | 15             |
| etc-mainnet-archivenode          | 16             |
| etc-mainnet-fullnode             | 17             |
| etc-mainnet-lightnode            | 18             |
| eth-mainnet-archivenode          | 19             |
| eth-mainnet-fullnode             | 20             |
| eth-mainnet-lightnode            | 21             |
| ltc-mainnet-fullnode             | 22             |
| ltc-mainnet-lightnode            | 23             |
| optimism-mainnet-fullnode        | 24             |
| osmosis-mainnet-fullnode         | 25             |
| polkadot-mainnet-fullnode        | 26             |
| polkadot-mainnet-lightnode       | 27             |
| polkadot-mainnet-archivenode     | 28             |
| polygon-mainnet-fullnode         | 29             |
| polygon-mainnet-archivenode      | 30             |
| sol-mainnet-fullnode             | 31             |
| thorchain-mainnet-fullnode       | 32             |
| bch-mainnet-unchained            | 33             |
| btc-mainnet-unchained            | 34             |
| bnb-mainnet-unchained            | 35             |
| bsc-mainnet-unchained            | 36             |
| gaia-mainnet-unchained           | 38             |
| doge-mainnet-unchained           | 39             |
| eth-mainnet-unchained            | 40             |
| avax-mainnet-unchained           | 41             |
| ltc-mainnet-unchained            | 42             |
| osmosis-mainnet-unchained        | 43             |
| thorchain-mainnet-unchained      | 44             |
| optimism-mainnet-unchained       | 45             |
| gaia-mainnet-grpc                | 46             |
| btc-mainnet-blockbook            | 47             |
| ltc-mainnet-blockbook            | 48             |
| bch-mainnet-blockbook            | 49             |
| doge-mainnet-blockbook           | 50             |
