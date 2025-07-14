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

## Arkeo Supported Services

Use either of these services for a comprehensive reference mapping between service names and their numeric identifiers, as used within Arkeo.

```
arkeod query arkeo all-services
```
```
http://localhost:1317/arkeo/services
```
