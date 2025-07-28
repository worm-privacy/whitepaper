# **WORM: A Privacy-Preserving, Cryptographically Scarce Token Backed by Ethereum Burn**

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

The WORM protocol consists of two main phases: ETH burn and WORM minting.

To initiate the process, a user first selects a secret value known as the `burnKey`. From this secret and the intended `receiverAddress`, the user generates a one-way ETH burn address by hashing the two values together using the Poseidon2 hash function and taking the first 20 bytes. The user then sends ETH to this derived address. Because the address is deterministically derived but not associated with any known private key, the funds are provably unrecoverable.

Once the ETH is burned, the user generates a zk-SNARK proof, called a **BETH proof**, that attests to the existence of this burn event in Ethereum’s global state. This involves verifying that an account existed in a specific Ethereum block’s state root, and that this account had a given balance of ETH and was located at a burn address matching the Poseidon-derived formula. The proof also ensures that a one-time-use `nullifier` is disclosed, preventing the same burn from being claimed multiple times.

The user may choose to allocate the resulting value into three components: a **fee** to a relayer, a **spend** amount that is immediately minted to the receiver, and a **remaining coin** that can be redeemed later with a separate proof. All of this is committed to a single public hash that is passed to the verifier smart contract, ensuring the integrity and privacy of the minting event.

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

## **7. Future Directions**

The WORM protocol may be extended in the future to support:

* Burn receipts from ERC-20 or ERC-721 assets.
* Multi-chain or Layer 2 issuance using cross-chain BETH proofs.
* Non-transferable “burn credentials” for proof of ETH sacrifice.
* Private donation, voting, or governance mechanisms tied to burns.

These extensions can expand WORM into a broader ecosystem for **cryptographically proven economic action**, where irreversible loss becomes the basis for novel value.

---

## **Conclusion**

WORM is a powerful new primitive for the Ethereum ecosystem. It enables users to convert irreversible ETH burns into usable, private, and scarce ERC-20 tokens — all without sacrificing composability or decentralization. By combining zero-knowledge proofs, proof-of-burn, and per-block issuance controls, WORM creates a new kind of value: one rooted in economic cost, privacy, and cryptographic verifiability.

Where previous privacy tools offered obscurity, WORM offers **plausible deniability**. Where synthetic assets relied on centralized bridges, WORM is grounded in ETH itself. In doing so, it redefines what it means to own a token that is truly backed by something — not collateral, not promise, but **irreversible action**.
