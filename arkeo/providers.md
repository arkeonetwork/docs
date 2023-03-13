# Data Providers

## Introduction
Data providers are the core of the arkeo network.  They provide access to api services that can be consumed by users, dapps, or any other system that requires decentralized access to arbitrary data. By registering themselves on chain, and providing a bond, they become discoverable on-chain for a client to consume their data. 

## Bonding
The first step for a provider to register their services with the Arkeo Network is for them to call `BondProvider`. In this call the provider will specify their public key, the service they are registering (e.g. `btc-mainnet-fullnode`) and a bond amount in ARKEO tokens. In order for clients to open a valid contract with a specific provider, the provider must post the minimum bond amount that is a configuration parameter of the arkeo network.

Providers are free to unbound at any time by calling `BondProvider` with a negative amount of ARKEO tokens representing the amount they wish to unbound.  If this drops their bound below the configured global minumum, clients will no longer be able to open new contracts with them. However, any existing contracts will remain valid until they expire or a user cancels. 

## Additional Provider Information
Providers are able to call `ModProvider` in order to establish or modify additional metadata about their services, state, and rates. The below fields are currently included as part of this call

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

## Provider payments
Payments for the consumption of services the providers offer are handled on-chain.  Depending on the [contract type](contracts.md) the client will either pay a subscription rate or a pay-as-you-go rate denominated in ARKEO tokens. For more information on the contract lifecycle, see [contracts](contracts.md).

## Sentinel
[Sentinel](../sentinel/sentinel.md) provides a reference implemention for how a provider may expose their services to ARKEO users and the needed cryptographic validation for each API requests. 