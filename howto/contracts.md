# Understanding Contracts with Arkeo Providers

Contracts in the Arkeo system represent agreements between service providers and their clients. These contracts define the terms of service, including payment terms, service duration, authorization methods, and rate limitations. They provide a structured and secure way to govern interactions, payments, and accountability between clients and providers.

Contracts play a critical role in Arkeo's decentralized transaction system by enabling off-chain authorization (client signatures and usage tracking) combined with on-chain settlement (payments processed through blockchain transactions). By structuring interactions this way, Arkeo ensures efficiency, transparency, and security while minimizing blockchain overhead.

## Contract Types Overview

### Arkeo supports two primary types of contracts:

- **Subscription (Fixed Duration)**: Suitable for fixed-term services, offering predictable billing and simple management.
- **Pay-As-You-Go (PAYG)**: Suitable for services charged per usage, offering flexibility and precise control over costs and resource use.

### Comparison of Contract Features

| Feature                    | Subscription (Less Strict) | Pay-As-You-Go (More Strict) |
|----------------------------|----------------------------|-----------------------------|
| Fixed Contract Duration    | ✅                          | ✅                           |
| Client Signatures Required | ❌                          | ✅                           |
| Rate Limiting              | ✅                          | ⚠️ (Optional)               |
| Prepaid Deposits           | ✅                          | ✅                           |
| Whitelist Management       | ✅                          | ⚠️ (Optional)               |
| Settlement Grace Period    | ✅                          | ✅                           |
| Deposit Reimbursement      | ❌                          | ✅                           |
| Strictness                 | ⚠️ (More Open)             | ⚠️ (More Strict)            |

## When to Use Each Contract Type

### Subscription (Fixed Duration)
Use this contract type when:
- Strict signatures is not desired.
- Whitelist authentication is acceptable.
- Service requirements and usage are predictable and stable.
- You prefer simple management with less frequent on-chain interactions.
- There is no need to track individual usage per transaction.

### Pay-As-You-Go (PAYG)
Use this contract type when:
- Services are highly variable or usage-based.
- Precise metering and billing per transaction are necessary.
- Enhanced control and accountability through client signatures are required.
- Better decentralization and security.
