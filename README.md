# [DRAFT] **WORM: A Privacy-Preserving, Cryptographically Scarce Token Backed by Ethereum Burn**

keyvankambakhsh@gmail.com | https://worm.cx

---

## **Abstract**

WORM is a novel ERC-20 token minted through the irreversible and private destruction of Ethereum (ETH). By leveraging zero-knowledge proofs and the emerging EIP-7503 standard for Private Proof-of-Burn, WORM enables users to convert ETH into a new, cryptoeconomically scarce asset without compromising their privacy or requiring changes to Ethereum's consensus rules. ETH is burned at one-way addresses derived from user-generated secrets, and a zk-SNARK proof attests to the validity of the burn. WORM is then minted on Ethereum by submitting this proof — without revealing which specific address was used or how much ETH was burned.

This mechanism introduces a new kind of cryptoasset: one that is scarce by construction, censorship-resistant by design, and private in issuance. The result is a token that can function as a private medium of exchange, a store of cryptographic value, and a composable primitive for the next generation of decentralized financial systems.

---

## **1. Introduction**

Tornado Cash demonstrated that cryptographic privacy alone is not enough. Despite using zero-knowledge proofs to unlink deposits and withdrawals, Tornado users were not immune to surveillance or enforcement, because their interactions with the protocol were publicly visible. The act of using Tornado was itself a signal — one that regulators and analytics firms could track, flag, and act upon. Privacy without **plausible deniability** is often just a red flag with better math.

WORM rethinks on-chain privacy from first principles. Rather than shielding transactions within a public system, it enables users to **privately exit Ethereum’s public accounting altogether**. By destroying ETH at a secret burn address and proving it in zero-knowledge, users mint WORM — a cryptographically scarce ERC-20 token backed by irreversible ETH loss, but issued anonymously.

WORM introduces **Private Proof-of-Burn**, a mechanism that proves ETH was burned in a canonical Ethereum block without revealing the burn address or transaction. No deposit contract is ever touched. The result is a shielded minting process, fully native to Ethereum, with no bridges, no custodians, and no identifiable footprint.

This architecture makes WORM more than just a new token — it serves as a **bridge from transparent Ethereum balances into a private asset class**. Users can move funds from their public ETH address into WORM and then into shielded systems like Railgun, Noir, or custom mixers, without revealing their source of funds or burning history.

Where Tornado offered unlinkability, WORM offers **invisibility**. Where others mask behavior, WORM erases it.

---

## **2. Architecture and Design**

The WORM protocol is built around a **two-token model** that decouples the private act of ETH burning from the public issuance of a scarce asset. These two tokens — **BETH** and **WORM** — serve distinct purposes within the system and interact through a carefully designed, time-based minting mechanism.

This separation of concerns allows the protocol to provide strong privacy guarantees, a predictable emission schedule, and a clean market for speculation and utility, all without centralized custody or bridges.

---

### **2.1 BETH: The Proof-of-Burn Receipt**

When a user burns ETH at a stealth address — derived from a secret `burnKey` and a chosen `receiverAddress` — they generate a **zk-SNARK proof** attesting that the ETH was irreversibly destroyed in a canonical Ethereum block. This proof is used to mint **BETH**, an ERC-20 token that serves as a **verifiable receipt of ETH burn**.

Each unit of BETH represents 1 ETH provably destroyed. However, BETH itself is not tied to a specific burn address or transaction in any observable way. The proof that produces BETH is privacy-preserving — it reveals only a single commitment hash derived from six fields (including the state root, nullifier, encrypted coin, and receiver address), without exposing the burn transaction, block number, or sender wallet.

Because BETH is transferable, users can **trade**, or **hold** their proof-of-burn rights. This opens up a rich design space:

* Burners can sell BETH on secondary markets without needing to interact with the WORM protocol.
* Market participants can acquire BETH and redeem it for WORM in future epochs.

This liquidity layer ensures that the protocol doesn’t force users to claim WORM directly after burning ETH. Instead, BETH becomes a **fungible claim token**, backed by irreversible ETH destruction and redeemable for WORM according to global supply rules.

---

### **2.2 WORM: A Cryptographically Scarce Token**

While BETH tracks ETH burned, **WORM tracks ETH scarcity**. WORM is an ERC-20 token minted through the consumption of BETH but governed by **strict issuance constraints** to preserve its value and deflationary properties.

WORM cannot be freely minted from BETH on a 1:1 basis. Instead, the protocol mints WORM on a fixed schedule: **50 WORM per epoch**, where each epoch spans **30 minutes**. Within each epoch, any BETH holder may submit a redemption proof to burn their BETH and claim a proportional share of the epoch’s WORM issuance.

This model creates a competitive redemption system:

* If few users redeem BETH in a given epoch, each receives a larger portion of the 50 WORM.
* If many users redeem BETH in the same epoch, each receives less.

The amount of WORM a user receives in an epoch is determined by their BETH share:

```
WORM received = (user’s BETH) / (total BETH redeemed in epoch) × 50
```

This structure creates a **dynamic incentive landscape**. Users must weigh their BETH redemption timing carefully — redeem early to avoid dilution, or wait for favorable conditions at the risk of increased competition.

Because only 50 WORM can be minted every 30 minutes (or 2,400 per day), the protocol introduces **artificial scarcity** overlaid on top of ETH burn scarcity. This gives WORM similar properties to Bitcoin or Ethereum post-merge: issuance is slow, predictable, and increasingly valuable as demand grows.

---

### **2.3 Privacy Preservation**

Despite being minted on-chain, **WORM preserves user privacy** by decoupling burn and mint events entirely.

Burning ETH happens through a stealth address derived from the Poseidon2 hash of a secret `burnKey` and receiver address. This burn address is indistinguishable from any normal Ethereum account and has no public link to the user. When the burn is proven via zk-SNARK, no information about the transaction, wallet, or block number is disclosed — only a cryptographic commitment.

This architecture ensures that:

* No one can link a WORM minter to an ETH burn.
* No one can determine how much ETH any WORM holder has destroyed.
* Users can mint or acquire WORM without ever interacting with a known privacy protocol.

The use of **BETH as a privacy-preserving receipt token** further reinforces this model. Since BETH is transferable, the entity that burns ETH need not be the one that mints WORM. This indirection — combined with the circuit’s use of nullifiers and encrypted balances — provides **plausible deniability** at every layer.

In practice, this means that **WORM acts as a bridge** from public ETH into a **private token** — not just in terms of unlinkability, but in terms of **optical invisibility**. No one can even prove that a wallet is a WORM participant unless that wallet publicly claims so.

### **2.4 Circom Circuits: Verifying the Proof-of-Burn**

At the heart of the WORM protocol is a zero-knowledge circuit written in Circom, which implements the **Private Proof-of-Burn** standard defined in [EIP-7503](https://eips.ethereum.org/EIPS/eip-7503). This circuit enables users to prove — without revealing any on-chain identifiers — that a given amount of ETH was irreversibly destroyed in a valid Ethereum block.

#### **Burn Address Derivation and Constraints**

The process begins with the user generating a secret `burnKey`, which serves as the private anchor for the burn. A corresponding Ethereum address, known as the **burn address**, is deterministically derived using the following steps:

1. **Poseidon2 Hashing**:  
   The user computes `Poseidon2(burnKey, receiverAddress)` where both inputs are 254-bit field elements.  
2. **Truncation to Ethereum Address**:  
   The first 160 bits of the result are used to form a standard Ethereum address.

Due to the truncation step, this mapping compresses a 508-bit input space into a 160-bit address space, implying potential collisions. To reinforce security, a **proof-of-work constraint** is introduced:

```

Keccak256(burnKey || receiverAddress || "EIP-7503") < 2^(232)

```

This condition requires that the hash output begins with three zero bytes (24 bits of difficulty), raising the effective search space from 2¹⁶⁰ to approximately 2¹⁸⁴, providing robust preimage resistance.

#### **Merkle Patricia Trie Proof**

Once ETH is sent to the derived burn address, the balance update is reflected in the Ethereum state tree. The circuit verifies this using a Merkle Patricia Trie (MPT) inclusion proof, confirming that:

- The derived burn address exists as a leaf in the global state trie.
- The account RLP at that leaf contains a nonzero ETH balance (indicating a completed burn).
- The trie root computed from the proof matches the canonical `stateRoot` of the Ethereum block in question.

This traversal is done by checking that the Keccak hash of each node correctly links to the next, ending in the expected `stateRoot`. The circuit essentially re-derives the Ethereum MPT path and asserts its correctness via cryptographic hashes.

#### **Public Inputs and Circuit Outputs**

The circuit exposes the following public inputs:

- `stateRoot`: The Ethereum block’s global state root.
- `balance`: The ETH balance held at the burn address (must be > 0).
- `nullifier`: A one-time-use commitment derived as `Poseidon2(burnKey, 1)` to prevent reuse.
- `coin`: An encrypted value `Poseidon2(burnKey, amount)` representing the burned amount for selective disclosure or partial spending.
- `commitment`: A combined hash representing the burn event and receiver address, used to authorize minting BETH or WORM.

These outputs are submitted as part of the zk-SNARK proof, allowing the WORM verifier contract to validate the ETH burn privately and securely, without ever revealing the underlying burn transaction or wallet address.

---

Together, this circuit forms the cryptographic foundation for WORM’s privacy-preserving, irreversible minting mechanism. It ensures that only genuine ETH burns — proven via Ethereum state — can result in the issuance of BETH and subsequently WORM, while preserving anonymity and preventing fraud or reuse.

---

## **3. Privacy and Plausible Deniability**

WORM introduces a form of **plausible deniability** that goes beyond traditional mixers like Tornado Cash or privacy-preserving wallets like Railgun. In those systems, users deposit funds into a shared pool and later withdraw them. While zero-knowledge proofs hide the link between deposit and withdrawal, both actions are public and attributable. This means that even if the connection is obscured, **the very act of using the system reveals intent**, making users vulnerable to surveillance, legal action, or censorship.

In contrast, WORM never reveals the ETH burn transaction. The burn happens in the open — anyone can see ETH being sent to an unknown address — but there is **no on-chain link** between that burn and the eventual minting of WORM. The zk-SNARK proof validates the burn in private, using only the block state root and a commitment hash. This **unlinkability** means that **no one, not even an adversarial observer, can determine which ETH transaction led to which mint**, or even whether the same user burned ETH at all.

Because the burn addresses are derived using a secret input (`burnKey`), and because the state root verification occurs entirely within the proof, **there is no need to register or deposit tokens publicly**, as is the case with mixers. The result is a **shielded minting process** that offers strong privacy by default — not merely unlinkability, but **plausible deniability**. A user could mint WORM and plausibly deny any association with a specific ETH burn event, because no link exists on-chain.

---

## **4. Comparison with Tornado Cash and Railgun**

Tornado Cash was a pioneering Ethereum mixer, using zk-SNARKs to obfuscate the link between deposits and withdrawals. However, the protocol relied on a fixed-size anonymity set and public deposits. Once a user interacted with Tornado Cash, it was clear they were attempting to hide funds — even if the link was cryptographically obscured. Moreover, since Tornado Cash involved direct ETH custody, it was eventually sanctioned and taken offline.

Railgun is a more generalized privacy wallet that supports shielded transfers across multiple assets. It uses advanced ZK tooling and offers a better user experience than Tornado Cash. However, it still operates within a shielded pool model, where user balances and actions are private, but **entry into the system is observable**. Additionally, Railgun does not involve economic destruction; tokens can be withdrawn, meaning privacy and scarcity are not linked.

In contrast, WORM offers:

* **Unlinkable entry**: Users burn ETH to derived addresses that are not marked or identified as WORM-related.
* **Zero custody**: ETH is not held or controlled by any contract or intermediary.
* **Irreversible issuance**: Once burned, ETH is gone forever — this economic cost backs WORM’s credibility.
* **Private minting**: WORM is minted from a zk-SNARK proof, not a withdrawal action.
* **Deniability**: There is no observable deposit or signal that someone will ever mint WORM.

This combination makes WORM not just private, but **plausibly deniable and censorship-resistant**.

---

## **5. Economic Model**

WORM is governed by a strict issuance model. Only users who provably destroy ETH via a valid BETH proof can mint WORM. The amount of WORM that can be created is strictly less than or equal to the ETH burned, taking into account optional fees and spends.

Additionally, WORM contracts enforce a **hard cap on how much can be minted per Ethereum block**, regardless of how many proofs are submitted. This cap protects against flash minting attacks and ensures that even in the face of mass burns, issuance remains predictable and gradual.

This makes WORM not only scarce due to ETH backing, but **scarce by construction**—independent of demand or speculation.

---

## **6. Implementation Details**

WORM is implemented using Circom circuits that verify Ethereum state root membership, Keccak and Poseidon hash functions, and Merkle Patricia Trie traversal. The BETH proof is generated off-chain using the burnKey, balance, and the relevant Ethereum block data. The WORM verifier smart contract receives the proof and commitment hash, and mints the specified amount of WORM.

The protocol adheres to [EIP-7503](https://eips.ethereum.org/EIPS/eip-7503), a proposed standard for Private Proof-of-Burn, which ensures composability with wallets and other ZK systems.

---

## **Conclusion**

WORM is a powerful new primitive for the Ethereum ecosystem. It enables users to convert irreversible ETH burns into usable, private, and scarce ERC-20 tokens — all without sacrificing composability or decentralization. By combining zero-knowledge proofs, proof-of-burn, and per-block issuance controls, WORM creates a new kind of value: one rooted in economic cost, privacy, and cryptographic verifiability.

Where previous privacy tools offered obscurity, WORM offers **plausible deniability**. Where synthetic assets relied on centralized bridges, WORM is grounded in ETH itself. In doing so, it redefines what it means to own a token that is truly backed by something — not collateral, not promise, but **irreversible action**.
