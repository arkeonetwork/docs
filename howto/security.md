# Arkeo Key Management and Signing for Subscribers

## Overview: Arkeo Key Management

Arkeo requires secure handling of cryptographic keys to sign nonce claims for off-chain authorization. Keys can be generated using the built-in arkeod command-line tool:

> arkeod keys add <Key-Name> --keyring-backend test

**Using the keyring-backend of test avoids password prompts, enabling automated processes to operate continuously. However, this convenience demands that you implement strong server security controls.**

**This method of using the test backend for the key management can be referred to as a "Hot Wallet."**

## Using the signhere Utility for Signing Nonces

Arkeo’s signing tool, signhere, provides a simple and secure method for signing nonce claims off-chain. Typically, signhere is invoked within automation scripts as follows:

> signhere -u Client-Key -m Message

## Recommended Security Practices:
- **Limit server and filesystem access to only trusted, authorized entities.**
- **Use dedicated, securely hosted servers with strong firewall rules and limited SSH access for any automated signing scripts.**
- **Regularly audit your scripts and key access logs to detect any unusual activities quickly.**

## Advanced Key Options:

For users or organizations with higher security needs, consider more robust key management solutions, which offer additional layers of protection while allowing automated signing:
- **External Hardware Security Modules (HSM)**
  - Securely stores private keys off-server.
  - Allows automated, secure signatures via authenticated sessions.
- **Cloud-based Key Management Service (KMS)**
  - Managed solutions like AWS KMS, Azure Key Vault.
  - Integrates with automated signing scripts through API calls.
- **Dedicated Local Key Daemon**
  - Runs independently and securely holds keys in memory.
  - Provides automated signing without repeated password prompts.

For optimal automated operation and performance, Arkeo currently recommends the straightforward use of keyring-backend=test, protected by robust server security measures, or more advanced options listed above if additional security layers are desired.