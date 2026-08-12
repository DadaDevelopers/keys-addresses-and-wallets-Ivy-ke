# Assignment 2: Bitcoin Address Types and HD Wallet Concepts

**Author:** Ivy Bett (Riri)
**Environment:** Bitcoin Core v30.3, built from source and running on testnet using Kali Linux

## 1. Generating Legacy, Bech32, and Bech32m Addresses

For this assignment, I used Bitcoin Core's `getnewaddress` command to generate three different types of Bitcoin addresses: Legacy, Bech32, and Bech32m. I specified the address type directly in the command for each one.

```bash
./build_dev_mode/bin/bitcoin-cli -testnet getnewaddress "" "legacy"
./build_dev_mode/bin/bitcoin-cli -testnet getnewaddress "" "bech32"
./build_dev_mode/bin/bitcoin-cli -testnet getnewaddress "" "bech32m"
```

### Addresses generated

| Address Type                  | Generated Address                                                | Prefix |
| ----------------------------- | ---------------------------------------------------------------- | ------ |
| Legacy (P2PKH)                | `mh2HxxvyJrB7ea3bWsRBKeqUcXsSZ4t1bC`                             | `m`    |
| Bech32 (native SegWit v0)     | `tb1q5mahzd0a6d46jd7cg7vjkfrm28jupw7xgjy28t`                     | `tb1q` |
| Bech32m (Taproot / SegWit v1) | `tb1pn747zat7n945pjp5lw5p7tet9a2khk608g338dvzm5y5pgsqgy4stvhslz` | `tb1p` |

### Understanding the three address types

**Legacy (P2PKH)** is the original Bitcoin address format. It uses a hash of a public key and, on the Bitcoin network, is associated with the traditional P2PKH locking script. On mainnet, Legacy addresses normally start with `1`, while testnet addresses can start with `m` or `n`.

**Bech32** was introduced with Segregated Witness (SegWit), which was activated in 2017. SegWit changed how signature information is stored in transactions. One of its major benefits was solving transaction malleability and reducing transaction fees because witness data is treated differently when calculating transaction weight. Bech32 addresses also use lowercase characters and have a strong checksum, which helps detect errors when an address is copied or entered incorrectly. On mainnet they normally start with `bc1q`, while on testnet they start with `tb1q`.

**Bech32m** is the address format associated with Taproot, which was activated in 2021. Taproot introduced SegWit version 1 and improved the way more complex spending conditions can be represented. For example, certain complex scripts can appear more similar to ordinary single-signature transactions on-chain, which can provide privacy and efficiency benefits.

Bech32m uses an updated checksum designed for SegWit version 1 and later. On mainnet, Taproot addresses normally start with `bc1p`, while on testnet they start with `tb1p`.

Overall, the three formats show how Bitcoin's address system has developed over time:

**Legacy → SegWit/Bech32 → Taproot/Bech32m**

Each upgrade addressed particular limitations and introduced improvements in areas such as transaction efficiency, security, privacy, and functionality.

---

## 2. Hardened vs Non-Hardened Keys

Modern Bitcoin wallets can use **Hierarchical Deterministic (HD) wallets**, where a single master key or seed can be used to generate many child keys. Instead of generating every key completely independently, the keys are organized into a structured tree using the BIP32 derivation system.

### Non-Hardened Derivation

A non-hardened derivation path looks like:

```text
m/0/1/2
```

With non-hardened derivation, child public keys can be derived using the parent's extended public key (xpub) and chain code.

There is an important security risk, however. If someone has the parent's extended public key and obtains one child private key, the parent private key can potentially be recovered. Once the parent private key is compromised, the other keys in that branch can also be derived.

This is important because xpubs can sometimes be shared for legitimate purposes, such as allowing someone to monitor a wallet without giving them the ability to spend the funds. If a corresponding child private key is leaked, however, the security of that branch can be seriously affected.

### Hardened Derivation

Hardened derivation uses paths such as:

```text
m/0'/1'/2'
```

The apostrophe (`'`) indicates that the derivation step is hardened.

Unlike non-hardened derivation, hardened child keys require the parent's private key. This removes the mathematical relationship that allows the parent private key to be recovered from an xpub and a leaked child private key.

Therefore, even if an xpub is exposed and a child private key from a hardened branch is leaked, the parent private key cannot be reconstructed using that information alone.

### Practical Importance

Hardened derivation is especially useful at important levels of a wallet's key hierarchy. For example, BIP44 uses hardened derivation for the purpose, coin, and account levels:

```text
m/44'/0'/0'
```

This helps prevent an exposed extended public key from creating a path back to higher-level private keys.

---

## 3. Why Deterministic Wallets Are Preferred Over Non-Deterministic Wallets

Before HD wallets became common, Bitcoin wallets could use a non-deterministic approach where each new address was generated from a separate random private key.

The main problem with this approach was backup management.

### Non-Deterministic Wallets

With a non-deterministic wallet, every new address could correspond to a completely independent private key. This meant that a backup made before generating new addresses might not contain the keys for those newer addresses.

For example, if a user backed up their wallet and then generated several new addresses, losing the wallet afterwards could mean that the older backup would not contain the private keys needed to recover funds received by those newer addresses.

This made regular backups very important and also made wallet recovery more complicated.

### Deterministic (HD) Wallets

HD wallets use a master seed from which the rest of the wallet's keys are derived. BIP32 defines the hierarchical key derivation structure, while BIP39 is commonly used to represent wallet seed material as a human-readable mnemonic phrase.

The main advantage is that a single properly stored backup can be used to recover the wallet's derived keys.

HD wallets also provide several other benefits:

* **Simpler backups:** One seed can be used to derive many wallet addresses.
* **Easier recovery:** The wallet can be restored on another compatible device using the required seed material.
* **Watch-only wallets:** An extended public key can be used to monitor derived addresses without giving access to the private keys.
* **Structured key management:** Derivation paths such as BIP44, BIP49, BIP84, and BIP86 allow wallets to organize keys for different address types and purposes.

### Conclusion

The main advantage of deterministic wallets is that they make key management much easier and more reliable. Instead of having to keep track of separate backups for individual private keys, an HD wallet can derive a large number of keys from a common seed.

From a security and usability perspective, this makes wallet backup and recovery much more practical and is one of the reasons HD wallets became the standard approach used by many modern Bitcoin wallets.
