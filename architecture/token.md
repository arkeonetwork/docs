# Arkeo Token

## Introduction

The **Arkeo Token (ARKEO)** is the native currency of the Arkeo Network. It serves several purposes:  
- Facilitates payments between clients and providers.  
- Acts as bonds posted by providers.  
- Used by validators to secure the network via Proof-of-Stake.  
- Grants voting rights for network governance.

> **Conversion:**  
> **1 ARKEO = 100,000,000 uARKEO**

---

## Maximum Supply

- **Total Supply:** 121,000,000 ARKEO  

---

## Airdrop

For details on the airdrop, refer to the [Airdrop Documentation](../airdrop.md).

---

## Reserve

The **Reserve** is a Cosmos module that holds the **non-circulating supply** of ARKEO. It is governed by chain rules rather than DAO or voting. Tokens are emitted from the reserve to validators and their delegates according to the **emission schedule**.

### Emission Formula:
```text
e = 6         # Emission curve
y = 6,311,520 # Blocks per year
r = Reserve Depth

block emission = r / e / y
```

This formula determines the number of tokens emitted per block, based on the reserve depth (`r`) and the emission curve (`e`).