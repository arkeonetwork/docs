# Arkeo Contracts

## Overview

Contracts in the Arkeo Network represent on-chain agreements between clients and data providers, defining the terms under which clients can access API services offered by the provider.

---

## Types of Contracts

Arkeo supports two types of contracts: **Subscription** and **Pay-as-You-Go**.

### Subscription Contracts

- **Purpose:** Designed for clients who prefer paying a fixed rate per block.  
- **Key Features:**  
  - Clients specify the contract duration in blocks and their `queries-per-minute` (QPM) to set an expected rate limit.  
  - Clients deposit an amount of ARKEO (or other IBC assets) calculated as:  
    **Deposit = Subscription Rate × Duration × QPM**  
  - The subscription rate is deducted per block while the contract remains active.  
  - **Cancellation:**  
    - Clients can cancel the contract at any time and receive a pro-rated refund for the unused deposit.  
    - Upon expiration or cancellation, the provider receives their full or pro-rated payment.  

---

### Pay-as-You-Go Contracts

- **Purpose:** Designed for clients who pay based on usage, with charges applied per API request.  
- **Key Features:**  
  - Clients specify a deposit amount from which charges are debited.  
  - API requests occur off-chain but require signed messages containing a nonce for authentication.  
  - Providers can submit these signed messages on-chain as proof of usage to debit the client’s deposit and credit their account.  
  - **Settlement Period:**  
    - After contract expiration, providers can make final claims within a settlement period (agreed upon during the `OpenContract` call).  
    - Once the settlement period ends, unused funds are returned to the client.  
  - **Cancellation Rules:**  
    - Clients can cancel pay-as-you-go contracts

---

## Delegated Contract Usage

- **Purpose:** Enables clients to delegate contract spending to another entity.  
- **How it Works:**  
  - The client specifies the public key of the delegate in the `Delegate` field of the `OpenContract` call.  
  - The delegate, if specified, or the client is referred to as the `Spender` of the contract.  
  - Useful for scenarios where a different private key is needed to make API requests.  

---

## Claiming Contract Income

- **Pay-as-You-Go Contracts:**  
  - Require a `ClaimContractIncome` call to reconcile off-chain usage proof with on-chain records.  
  - Anyone can submit these claims on behalf of the provider.  
  - The provider is credited the amount specified by the usage, while the client is debited from their deposit.  

---

## Contract Uniqueness

- At any given time, a single client (or delegate) can only hold **one active contract** with a specific provider for a particular API service.  