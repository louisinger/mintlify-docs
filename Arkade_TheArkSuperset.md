# **Arkade: The Ark Superset**

## **Operator Collusion Mitigation**

To mitigate the risk of operator collusion within the Ark protocol's offchain execution engine, the following strategies and technological safeguards are employed:

### **Separate Signing Entity: Ark Signer**

* The Ark Operator signing key is allocated to a dedicated and independent entity, the Ark Signer.  
* Ark Signer is exclusively responsible for co-signing Ark transactions.  
* It does not manage boarding addresses nor participate in tree signing.

### **Trusted Execution Environment (TEE)**

* Ark Signer operates within a Trusted Execution Environment (TEE), providing hardware-level security assurances.  
* The Ark Signer key is generated and resides within the TEE, ensuring even the Ark Operator has no direct access to it.

### **Remote Attestation**

* Using open source and reproducible builds, TEE remote attestation verifies that Ark Signer software is genuine, honest, and tamper-free.  
* Attestation results provide transparency and build trust in the Ark Signer's operational integrity.  
* If the TEE runs different software, the Ark Signer key is lost. While initial remote attestations verify software integrity, the key's continued operational activity serves as ongoing proof of integrity.

### **End-to-End Encrypted Communication**

* Ark Operators act as gateways to Ark entities like the Ark Signer. There exists a potential risk of user communication being censored by Operators.  
* Introducing end-to-end (E2E) encryption ensures that communication between users and Ark Signer is confidential.  
* Only users and the Ark Signer can decrypt the content of requests and responses, such as Ark transaction signatures or proof presentations. The Ark Operator remains oblivious to the actual content, preventing targeted censorship.

### **Self-Deployed TEEs**

* Users may deploy their own Trusted Execution Environments (TEE) with the same Ark Signer software.  
* Original Ark Signer can perform handshake, verification, and replication with self-deployed TEEs.  
* Self-deployed Ark Signers allow users to bypass Ark Operators as gateways while still relaying essential data from the Ark Operator to prevent double-spending.

### **Commitment Against Double Spending**

* An Ark Signer explicitly commits never to double-spend Virtual Transaction Outputs (VTXOs).  
* It refuses to sign conflicting Ark transactions that would result in double-spending.

### **Exclusion of Forfeit Transactions**

* The commitment to avoid double-spending excludes forfeit transactions, which become invalid if a round fails.

### **Enforcement via Reputation and Capital Risk**

* Ark Signers uphold their commitments via dual enforcement mechanisms:  
  * **Reputation**: A critical resource built through consistent adherence to their commitments.  
  * **Capital Risk**: Financial collateral locked to the Ark Signer’s key, which can be penalized in the event of misconduct.

### **Proof and Capital Burn**

* Violations can be conclusively demonstrated by presenting two conflicting transactions signed by the same Ark Signer.  
* Proof of violation triggers severe repercussions, including:  
  * Burning the Ark Signer’s reputation.  
  * Immediate forfeiture (burn) of the Ark Signer’s capital.

### **Capital Burn Incentive**

* Due to TEEs' inability to reliably store or access consistently updated data, Operators may attempt to coerce Ark Signers into double-spending through data corruption or withholding.  
* Implementing a capital burn penalty disincentivizes malicious actions by Operators, enhancing overall system integrity and trustworthiness.

These measures collectively ensure the robust security and integrity of the Ark protocol against operator collusion and other malicious behaviors.

## **Arkade Script: Enhanced Bitcoin Scripting**

With the foundational integrity and guarantees established by the Ark Signer, Arkade introduces new programmable capabilities to expand the utility of Ark Transactions.

* Arkade Script is an expanded Bitcoin Script interpreter running within the Ark Signer.  
* Expands Bitcoin operation codes with introspection, arithmetic, logic, and asset operations.  
* Users create Arkade scripts, hash them, and tweak the Ark Signer's key in outputs, enabling commitment to enhanced scripting while maintaining Bitcoin unilateral exit capability.  
* As part of user requests for Ark transaction signatures, additional Arkade execution data can be provided: a virtual witness and a virtual script for every VTXO.  
* The Ark Signer verifies that the script computes correctly to match its tweaked key, executes with the provided witness, and cosigns upon successful verification.  
* Ark Signer provides both a signature on the Ark transaction and a signed proof of execution for the entire request, ensuring verifiable honest execution.

## **Ark Batcher: Constructing Fair and Transparent Batches**

By decoupling batch construction from the Ark Operator, the Ark Batcher brings fairness, transparency, and scalability to the Ark protocol. This modularity helps ensure participants can trust the selection and ordering of their intents without bias or manipulation.

* The Ark Batcher is responsible for collecting intents and constructing batches based on defined policies.  
* Operates within a Trusted Execution Environment (TEE) to ensure transparent, honest selection of transaction intents.  
* Policy-driven properties such as offered fees dictate intent ordering.  
* The Ark Batcher engages with Ark Liquidity Providers (LPs) to determine the requirements (e.g., sufficient fees) for intents to be adequately funded.

## **Ark Liquidity Providers: Funding Ark Batches**

Unlocking external liquidity sources and inviting broader market participation, Ark Liquidity Providers (LPs) reduce dependence on the Ark Operator and enable a more decentralized and sustainable funding layer for Ark rounds.

* Ark Batches are funded by Ark Liquidity Providers (LPs) via onchain wallets offering funding bids to the Ark Batcher.  
* A single batch may be funded by multiple LPs.  
* The expiry branch of a batch is co-signed by all LPs who contribute funds.  
* To avoid fund lockups, a precomputed Ark sweep transaction is created, which splits the funded amount proportionally based on each LP’s contribution.  
* This sweep transaction ensures that no LP is left with frozen or unclaimable funds.


