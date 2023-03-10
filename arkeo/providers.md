# Data Providers

## Introduction
Data providers are a critical element in the arkeo network.  They provide access to api services that can be consumed by users, dapps, or any other system that requires decentralized access to arbitrary data. By registering themselves on chain, and providing a bond, they become discoverable for a client to being consuminbg their data service. 

## Bonding
The first step is for a provider to register a service with the Arkeo Network by calling `BondProvider`. In this call the provider will specify their public key, the name of the service they are registering (e.g. `btc-mainnet-fullnode`) and a bond amount in ARKEO tokens. In order for clients to open a valid contract with a specific provider, the provider must post the minimum bond amount specified by the arkeo network.

Providers are free to unbond at any time by calling `BondProvider` with a negative amount of ARKEO tokens representing the amount they wish to unbond.  If this drops their bond below the configured global minumum, clients will no longer be able to open new contracts with them. However, any existing contracts will remain valid until they expire or a user cancels. 

## Additional Provider Information
Providers call `ModProvider` in order to establish or modify additional metadata about their services, state, and rates. The below fields are currently included as part of this call

```go
Service             string                                    
MetadataUri         string                                    
MetadataNonce       uint64                                    
Status              ProviderStatus                            
MinContractDuration int64                                     
MaxContractDuration int64                                     
SubscriptionRate    int64                                     
PayAsYouGoRate      int64                                     
SettlementDuration  int64    
```

This information is stored on-chain and can be queried by clients in order to determine the best provider for their needs. Additionally the [directory service](../directory/directory.md) will be able to query this information in order to provide a more complete view of the providers available on the network.

## Provider Payments
Payments for the consumption of services the providers offer are handled on-chain.  Depending on the [contract type](contracts.md) the client will either pay a subscription rate or a pay-as-you-go rate denominated in ARKEO tokens. For more information on the contract lifecycle, see [contracts](contracts.md).

## Sentinel
[Sentinel](../sentinel/sentinel.md) provides a reference implemention for how a provider may expose their services to ARKEO users and the needed cryptographic validation for each API requests. 