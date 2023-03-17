# Arkeo Contracts

## Introduction
Contracts represent an on-chain agreement between a client and a provider outlining the terms by which the client can consume API services offered by the provider.

## Creating a contract
A client, such as a dApp, can open a contract for their own use or delegate the ability to spend the contract deposit to a different client. Once a client has identified a provider that they wish to utilize, they call `OpenContract` specifying the parameters of the contract they would like to open.

```go
Provider           github_com_arkeonetwork_arkeo_common.PubKey
Service            string
Client             github_com_arkeonetwork_arkeo_common.PubKey
Delegate           github_com_arkeonetwork_arkeo_common.PubKey
ContractType       ContractType
Duration           int64
Rate               int64
Deposit            github_com_cosmos_cosmos_sdk_types.Int
SettlementDuration int64
```

## Contract types
Contracts can be of two types: subscription, or pay-as-you-go. Subscription contracts are for clients that wish to pay a fixed rate per block. Pay-as-you-go contracts are for clients that wish to pay a fixed rate for each API request they make.

### Subscription contracts
Clients who would like to open a subscription contract must specify the duration of the contract in blocks. The client will deposit the amount of ARKEO
tokens equal to the subscription rate multiplied by the duration of the contract. The client will be charged the subscription rate for each block that the contract is active. Subscription contracts can be cancelled at any time by the client. The client will be refunded the remaining pro-rated deposit and the contract will be closed. Upon expiration or cancellation of the contract, the provider will receive their full or pro-rated payment.

### Pay-as-you-go contracts
Clients who would like to open a pay-as-you-go contract will specify a deposit amount that is debited from their usage. The client will be charged the pay-as-you-go rate for each API request they make. API requests are made off chain, but require the client to sign a message which includes a nonce to authenticate their request. These signed messages can be submitted on-chain as a proof of usage at any time and will accordingly debit the deposit amount of the Client and credit the provider. Pay-as-you-go contracts also entail an optional settlement period after the contract expires during which the final claims against a deposit can be made before unused funds are returned to the client. The duration of this settlement period is specified by the provider and agreed to in the `OpenContract` call by the client.

## Delegating the use of a contract
Clients can delegate the ability to spend the contract deposit to a different client. This is useful for clients who wish to allow a third party to make API requests on their behalf. The client can delegate the contract to a different client by specifying the public key of the client they wish to delegate to in the `Delegate` field of the `OpenContract` call. We refer to the delegate if it exists or the client as the `Spender` of the contract.

## Claiming income from a contract
Pay-as-you-go contracts require a call to `ClaimContractIncome` in order for the off-chain proof of usage to be reconciled on chain. Anyone can submit these claims on behalf of the provider. The provider will be credited the amount of ARKEO tokens specified by the contract usage and the client will be debited the same amount from their deposit.

```go
ContractId uint64
Spender    github_com_arkeonetwork_arkeo_common.PubKey
Signature  []byte
Nonce      int64
```

## Contract uniqueness
At any one time, a single client (or delegate) can only have one open contract against a provider for a specific API service.