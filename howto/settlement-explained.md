# The Arkeo Settlement Process

In Arkeo, settlement is a crucial phase where providers are paid for the claims they've accumulated. Providers collect client-signed claims (containing nonces and usage data) off-chain and periodically submit these claims to the Arkeo blockchain. This submission initiates the decentralized reconciliation and payment process.

## Importance of the Settlement Duration

The settlement duration is the specific time window (measured in blocks) allocated to providers for submitting claims after the contract has expired. It is critical because:

- **It ensures timely payments and maintains accurate accounting on the blockchain.**
- **It encourages providers to regularly reconcile and submit their claims, enhancing overall system efficiency and reliability.**

## How Providers Get Paid

Providers receive payments by submitting collected claims within the settlement duration to the Arkeo blockchain. Upon successful submission:

- **The blockchain verifies the claims, checking nonces, signatures, and contract terms.**
- **The decentralized settlement script on the blockchain reconciles these claims.**
- **Once verified, the claimed amount of tokens is automatically transferred from the locked contract deposit directly to the provider's account.**

## Decentralized Reconciliation and Settlement

The settlement script on the blockchain is invoked automatically when providers submit their claims. The process involves:

- **Validating nonce sequences and verifying client signatures.**
- **Ensuring submitted claims match previously agreed contract terms, including rates and allowed durations.**
- **Executing token transfers to settle payments transparently and securely.**

This decentralized mechanism eliminates the need for third-party intermediaries and ensures trustless transactions, making it a robust solution for metered billing.

## Consequences of Missing the Settlement Duration

If a provider fails to submit their claims within the allocated settlement window:

- **Claims become invalid and cannot be submitted afterward.**
- **Tokens reserved in the contract for these unsettled claims remain locked until the contract expires, after which any unclaimed tokens revert to the client's account.**
- **Providers lose potential earnings, underscoring the importance of timely and accurate settlement submissions.**

Providers are advised to implement reliable Sentinel services and maintain timely claim submissions to avoid missed settlements and associated revenue loss.