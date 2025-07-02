# Arkeo Frequently Asked Questions
Here are some commonly asked questions. We will update these as more requests come in.

### What is Arkeo?
Arkeo is a decentralized ecosystem enabling trustless, pay-as-you-go transactions. It combines off-chain authorization (client-signed claims) with on-chain settlement, providing efficient, transparent, and secure payment processing.

### How does Arkeo handle payments?
Arkeo uses off-chain authorization with on-chain settlement. Clients authorize usage off-chain through signatures, which providers periodically submit to the blockchain for secure and decentralized payment reconciliation.

### What contract types does Arkeo support?
Arkeo supports two contract types:
- **Subscription Contracts:** Fixed-term, predictable billing.
- **Pay-As-You-Go (PAYG) Contracts:** Usage-based billing requiring client signatures per usage event.

### How does Arkeo ensure security in key management?
Arkeo provides secure key management options, from simple test setups for automation (password-free signing) to advanced integrations like hardware security modules (HSMs) or external key management services (KMS).

### What happens if a provider doesn’t settle within the specified settlement duration?
If a provider fails to submit claims for settlement within the defined settlement duration, those claims expire and the provider forfeits payment for the related usage events.

### How are client signatures verified by providers?
Providers verify client signatures using the Arkeo blockchain’s cryptographic functionality. Every claim includes a unique nonce and client signature, ensuring authenticity and preventing replay attacks.

### Can a contract be delegated for third-party management?
Yes, Arkeo contracts support optional delegation. Providers or subscribers can assign a delegate by specifying the delegate’s public key when creating a contract, allowing trusted third parties to manage or settle claims.

### How do rate limits work within Arkeo contracts?
Arkeo contracts can optionally define a maximum queries-per-minute (QPM) parameter to enforce rate limits. This protects providers from excessive requests, especially valuable for PAYG contracts.

### What tools does Arkeo offer to simplify nonce signing?
Arkeo provides the `signhere` command-line tool for signing nonces and usage data quickly and securely. It enables automated signing processes without manual intervention or password entry when using appropriate key management practices.

### What happens if the Sentinel process misses blockchain events?
If Sentinel becomes desynchronized or misses blockchain events (due to downtime or connection issues), it can be re-synced by restarting or reindexing events. Regular Sentinel maintenance is recommended to ensure claims are accurately tracked and settled.

### Do I have to be a validator or run my own full node to be a provider?
No. Any funded Arkeo account can bond and run a provider service. For convenience, you only need the CLI binaries, but using an external node may impact your reliability and settlement guarantees.