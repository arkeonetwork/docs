# Arkeo Network Documentation
Arkeo is a free-market decentralized network for providing access to
blockchain data. Its intended goal is to create a decentralized option for
interacting with any blockchain (although not limited to) and its data. It is somewhat analogous to
[Infura](https://infura.io/) and [Alchemy](https://www.alchemy.com/), which
today are the predominate providers in the industry for blockchain data. Both
of these providers are highly centralized which creates a problem for web3 to
be self-reliant and not require web2 to function. In addition these provider
can censor users access to specific services while also do not have to respect
privacy.

## The Arkeo Solution
The Arkeo solution, at a high level, entails three points points.
 1) **Free market data providers** - anyone can be a data provider and provide
data for any blockchain. Each data provider can choose their own pricing,
which allows the free market to discover the price of blockchain data.
 1) **Trustless payment solution** - a user can pay a data provider in any
IBC-enabled asset (will expand to other asset later). This includes the
ability to make micro-payments allowing more flexibility for users.
 1) **On-chain reputation** - since all relationships between users and data
providers are public, it is easy to see the reputation of any data provider.
This helps inform the community about which data providers are high quality vs
low quality

## How It Works
Arkeo can be explained in how it in works in five simple bullet points.
 * Individuals can run full nodes of any blockchain and allow people to query their node(s) at a price of their choosing. These are called data providers.
 * Users can open a contract on-chain with specific data providers and escrow tokens. Contracts can be either subscription based or pay-as-you-go, with more options coming.
 * Each query between user and data provider is cryptographically provable and redeemable on-chain as income for data providers.
 * 10% of data provider income is paid to the network reserve, which is used to emit tokens to validators as block rewards.
 * Data provider reputation can be established by on-chain contract data such as provider age, user retention rates, income, etc. From this, one can abstract data provider quality.
