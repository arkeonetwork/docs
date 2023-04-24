# Arkeo Token

## Introduction
The Arkeo Token (ARKEO) is the native currency of the Arkeo Network. It used to facilitate payments between clients and providers, for bonds posted by providers, and by validators for securing the network via proof-of-stake. Additionally, it confers voting rights for governance of the network.

## Max Supply
Total Supply: 121,000,000 ARKEO

## Airdrop
See [Airdrop](../airdrop.md)

## Reserve
The reserve is a cosmos module that holds non-circulating supply of ARKEO. It
is governed by chain rules rather than any DAO/voting. Token are emitted from
here to validators and their delegates using the emission schedule.

```
e = 6 # emission curve
y = 5256666 # blocks per year
r = reserve depth
block emission = r / e / y
```
