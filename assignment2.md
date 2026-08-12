# Assignment 2: Bitcoin Address Types and HD Wallet Concepts

**Author:** Ivy Bett 
**Environment:** Bitcoin Core v30.3, built from source, running on testnet (Kali Linux)

---

## 1. Generating Legacy, Bech32, and Bech32m Addresses

Generated using `getnewaddress` with each address type specified:

```bash
./build_dev_mode/bin/bitcoin-cli -testnet getnewaddress "" "legacy"
./build_dev_mode/bin/bitcoin-cli -testnet getnewaddress "" "bech32"
./build_dev_mode/bin/bitcoin-cli -testnet getnewaddress "" "bech32m"
```

### Actual generated addresses

| Type | Address | Prefix |
|---|---|---|
| Legacy (P2PKH) | `mh2HxxvyJrB7ea3bWsRBKeqUcXsSZ4t1bC` | `m` |
| Bech32 (native SegWit v0) | `tb1q5mahzd0a6d46jd7cg7vjkfrm28jupw7xgjy28t` | `tb1q` |
| Bech32m (Taproot / SegWit v1) | `tb1pn747zat7n945pjp5lw5p7tet9a2khk608g338dvzm5y5pgsqgy4stvhslz` | `tb1p` |

### What each format is and why it exists

- **Legacy (P2PKH)** — the original Bitcoin address format from 2009. Encodes a hash of a public key (`OP_DUP OP_HASH160 <PubKeyHash> OP_EQUALVERIFY OP_CHECKSIG`, see Assignment A). On mainnet these start with `1`; on testnet, `m` or `n`.

- **Bech32 (native SegWit v0)** — introduced with the SegWit upgrade (activated 2017), which restructured how signature data is stored in a transaction. This fixed a technical bug called transaction malleability and lowered fees, since witness data is counted differently for fee purposes. Bech32 addresses use lowercase-only encoding with a much stronger built-in checksum, making manual copy errors far more likely to be caught before funds are sent. Mainnet prefix `bc1q`, testnet `tb1q`.

- **Bech32m (Taproot / SegWit v1)** — introduced with the Taproot upgrade (activated 2021). Taproot allows complex spending conditions (e.g. multisig, HTLCs) to look identical on-chain to a simple single-signature payment, improving both privacy and efficiency. Because Taproot introduced a new address *version*, developers found the original bech32 checksum math had a subtle weakness outside version 0, so bech32m — a corrected checksum — was designed specifically for it. Mainnet prefix `bc1p`, testnet `tb1p`.

Each format represents a real, sequential upgrade: Legacy → SegWit (fixed malleability, cut fees) → Taproot (improved privacy/efficiency for complex scripts).

---

## 2. Hardened vs Non-Hardened Keys

Modern wallets don't store a separate random key per address — they use **HD (Hierarchical Deterministic)** structure: one master key generates a whole tree of child keys via a defined formula (BIP32).

### Non-Hardened Derivation
- Path notation: `m/0/1/2...`
- Child keys are derived using the parent's **public** key plus a chain code.
- **Risk:** if someone has the parent's extended public key (xpub) *and* just one leaked child private key, they can mathematically reconstruct the **parent private key** — and from there, derive every other child key in that branch.
- This matters in practice: xpubs are often shared for legitimate reasons (e.g. watch-only monitoring for an accountant or exchange). A single leaked child key would then compromise the entire branch.

### Hardened Derivation
- Path notation: `m/0'/1'/2'...` (marked with an apostrophe, sometimes written `h`)
- Child keys are derived using the parent's **private** key instead of the public key.
- This breaks the mathematical relationship exploited above — even with the xpub and a leaked child private key, the parent private key **cannot** be reconstructed.

### Practical implication
Standards like BIP44 structure derivation paths with hardened steps at the account level (e.g. `m/44'/0'/0'`), keeping account-level keys safe even if an xpub for a specific branch is exposed elsewhere.

---

## 3. Why Deterministic Wallets Are Preferred Over Non-Deterministic Wallets

### Non-Deterministic Wallets (legacy approach, pre-2013 Bitcoin Core default)
- Every new address is generated from an independent, randomly generated private key with no relationship to previous keys.
- Required **constant backups** — each new address is a new key not covered by any prior backup.
- If a wallet file was lost after new addresses were generated, funds sent to those new addresses were **permanently unrecoverable**, even with an older backup on hand.

### Deterministic (HD) Wallets (BIP32 standard)
- All keys are derived from a **single seed**, typically a 12 or 24-word mnemonic phrase (BIP39).
- **One backup covers all future keys** — including addresses not yet generated at backup time.
- Simplifies **recovery**: a wallet can be fully restored on a new device using only the seed phrase.
- Enables **watch-only wallets**: an xpub can be shared to monitor incoming transactions across all derived addresses without exposing any private keys.
- Supports **organized key structures** via standardized derivation paths (BIP44/49/84/86), letting wallets separate keys by purpose (legacy, SegWit, Taproot, different accounts, etc.).

### Conclusion
Deterministic wallets turn "back up every single key, forever, after every new address" into "back up once, and every future key is already covered." This is both a stronger security guarantee for users and a much simpler foundation for wallet developers to build reliable backup and recovery experiences on.
