## Product Requirements Document: Granite Ridge - Core

**Granite Ridge** is a public-facing verification gateway that enables anyone to validate the authenticity of a digital document by cross-referencing it against decentralized attestation records on the Sui blockchain.

---

### 1. Executive Summary
* **Product Name:** Granite Ridge
* **Mission:** To provide a trustless, transparent interface for document verification, bridging the gap between enterprise e-signatures and decentralized ledgers.
* **Core Value Prop:** Eliminates "document fraud" by allowing third parties to verify that a file in their possession is the exact version signed by a specific entity.

---

### 2. User Workflow & Architecture

| Stage | Action | Technology |
| :--- | :--- | :--- |
| **1. Execution** | Document is signed by parties. | Zoho Sign (or similar) |
| **2. Processing** | Workflow engine triggers on completion. | n8n |
| **3. Storage** | Document is stored as a blob. | Walrus (Decentralized Storage) |
| **4. Attestation** | Transaction recorded: `(timestamp, attester_wallet, doc_hash)`. | Sui Blockchain |
| **5. Discovery** | Public user identifies the signer's wallet via DNS. | DNS TXT Records |

---

### 3. Functional Requirements

#### 3.1 Verification Interface
* **File Uploader:** Drag-and-drop zone for the user’s digital document.
* **Hash Generation:** The system must generate a $SHA\text{-}256$ hash of the uploaded file locally (client-side) to maintain privacy before querying the ledger.
* **Domain Input:** A field for the user to enter the claiming organization's domain (e.g., `beechfork.net`).

#### 3.2 Discovery Logic (The DNS Bridge)
* The system shall perform a DNS lookup for `_attester.[domain]`.
* It must parse the TXT record to extract:
    * `chain_id`: To ensure the query hits the correct network.
    * `attester`: The hex-encoded Sui wallet address.
    * `revoked`: A boolean check to see if the gateway is currently active.

#### 3.3 Blockchain Query
* Query the Sui ledger for an object matching the $doc\_hash$ and $attester\_wallet$.
* **Success State:** Display "Verified" with the timestamp and a link to the Sui explorer.
* **Failure State:** Display "Authenticity Unconfirmed" if the hash/wallet combo does not exist.

---

### 4. Security & Risk Mitigation
**Concern:** DNS Spoofing/Poisoning. An attacker could intercept a DNS request and return a malicious TXT record containing their own wallet address, making a forged document appear "verified."

#### Mitigation Strategies (Security Specialist Input)
To harden Granite Ridge against DNS-based attacks, we will implement a multi-layered defense:

* **1. Mandatory DNSSEC Validation:**
    The backend (or client-side resolver) must strictly require **DNSSEC** (Domain Name System Security Extensions). If the records are not cryptographically signed by the domain owner, the verification should be flagged as "Low Confidence" or blocked.
* **2. The "On-Chain Registry" Fallback:**
    Instead of relying solely on DNS, maintain a "Verified Signers" smart contract on Sui. DNS acts as a pointer, but the system cross-references the $wallet \rightarrow domain$ mapping against a registry managed by a consortium or trusted DAO.
* **3. Multi-Path Discovery:**
    Query multiple DNS providers (e.g., Google, Cloudflare, Quad9). If there is a mismatch in the returned TXT records, trigger a "Security Alert" and halt verification.
* **4. `well-known` HTTP Fallback:**
    In addition to DNS, look for a JSON file hosted at `https://[domain]/.well-known/granite-bridge.json`. Since this requires a valid SSL/TLS certificate, it is significantly harder to spoof than a raw DNS record.

---

### 5. Success Metrics
* **Verification Latency:** < 3 seconds from file upload to ledger confirmation.
* **Trust Accuracy:** 0% false positives (authenticated documents must match the ledger exactly).
* **Uptime:** 99.9% availability of the verification gateway.

---

> **Technical Note:** All document hashes are generated using the $SHA\text{-}256$ algorithm to ensure compatibility between Zoho Sign's output and the Sui transaction data.
