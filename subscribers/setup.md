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

## Create a Hot Wallet (Automated Key)

Providers automate claim settlements using a dedicated "hot wallet":

```
arkeod keys add <provider-hot-wallet> --keyring-backend test
```

- Fund this wallet only with minimal amounts required for bonding and fees.

> Security Notice:
> - The test backend stores your keys unencrypted; use it only for automated scripts.
> - Never use this wallet for significant funds.

## Arkeo Supported Services

Use either of these services for a comprehensive reference mapping between service names and their numeric identifiers, as used within Arkeo.

```
arkeod query arkeo all-services
```
```
http://localhost:1317/arkeo/services
```

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