# Subscriber Initial Setup Guide

Connecting securely and reliably to Arkeo provider services requires thoughtful initial setup. This guide covers key considerations, security practices, and the provider discovery process to ensure you get started correctly.

## Initial Considerations and Security

### Key Security

- Always safeguard your private keys; never share them publicly.
- For automation and script interactions, use a dedicated keyring backend (`test`). Refer to the [Security Documentation](#) for details on safe key management practices.
- Regularly back up your keys securely to prevent data loss.

## Selecting the Right Service

Determine clearly which service type you require and confirm its numeric identifier from the service mapping chart below. Correct service selection ensures compatibility and reliability.

### Provider Discovery

Use the following command to find reliable providers offering your chosen service type:

```
arkeod query arkeo list-providers --output json | \
jq '.provider[] | select(.status=="ONLINE" and .service==<service-number>)'
```

Replace <service-number> with the numeric identifier corresponding to your chosen service.

### Provider Evaluation

Before choosing a provider, carefully check:
- Provider metadata (accessible via their advertised metadata_uri).
- Contract terms such as rate, minimum duration, and settlement details.
- Provider reputation and reliability based on community feedback.

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

## More Information:
Once you’ve completed the initial subscriber setup, explore these additional documentation pages to fully prepare and manage your Arkeo contracts:
- [Subscription Contract Setup](setup-subscription.md)
  - Step-by-step instructions to create, configure, test, and close predictable and consistent subscription contracts.
- [Pay-As-You-Go (PAYG) Contract Setup](setup-payg.md)
  - Guide for flexible, usage-based contract creation and management, perfect for dynamic usage patterns.
- [Contract Management](contract-management.md)
  - Detailed guidance and scripts to automate and ensure uninterrupted service through proactive contract handling.
- [Monitoring](monitoring.md)
  - Best practices and quick commands to keep an eye on service health, active contracts, and your hot wallet balances.

Utilizing these resources ensures robust, secure, and continuous access to Arkeo provider services.