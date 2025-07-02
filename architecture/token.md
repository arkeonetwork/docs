# Arkeo Token

## Introduction

The **Arkeo Token (ARKEO)** is the native cryptocurrency of the Arkeo Network, playing a vital role in the ecosystem by serving as:

- A means of payment between subscribers (clients) and data providers.
- Bonds posted by providers to demonstrate commitment and credibility.
- A staking asset used by validators and delegators to secure the network through Proof-of-Stake consensus.
- A governance token enabling holders to participate in voting and decision-making processes for network upgrades and changes.

- **Conversion:**  
  - **1 ARKEO = 100,000,000 uARKEO**

## Maximum Supply

- **Total Supply:** 121,000,000 ARKEO

This fixed supply helps maintain predictable economics, transparency, and long-term sustainability.

## Airdrop

Detailed information about the airdrop process is available in the [Airdrop Documentation](airdrop.md).

## Reserve

The **Reserve** is a Cosmos module managing the **non-circulating ARKEO supply**. It operates based on predefined network rules, independently from DAO governance or direct voting. Tokens from the reserve are systematically released to validators and delegators as rewards according to a controlled emission schedule.

### Emission Formula

Token emissions per block are determined by the following formula, designed to ensure predictable inflation and sustainable network incentives:

```text
e = 6         # Emission curve
y = 6,311,520 # Blocks per year
r = Reserve Depth

block emission = r / e / y
```

This formula determines the number of tokens emitted per block, based on the reserve depth (`r`) and the emission curve (`e`).

## Staking and Rewards

ARKEO tokens can be staked with validators to secure the network and earn rewards. Validators operate nodes that validate transactions, produce new blocks, and maintain network security. Delegators contribute by delegating their ARKEO tokens to validators, sharing in the earned rewards proportional to their stake.

Staking ARKEO tokens helps maintain network security and decentralization, rewarding participants who actively support network integrity. For step-by-step instructions, refer to the Validator Documentation.

## Governance Participation

ARKEO token holders actively shape the network’s future by participating in governance. Token holders can vote on important proposals such as:
- Network upgrades and changes.
- Adjustments to economic parameters and emission schedules.
- Introduction of new features and integrations.

Engaging in governance empowers the community, enhances decentralization, and ensures decisions reflect the interests of network participants. Active governance fosters a robust, responsive, and community-driven ecosystem.