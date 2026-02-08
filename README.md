# Adamant 💎🔐

![Rust](https://img.shields.io/badge/Language-Rust-orange)
![Compliance](https://img.shields.io/badge/Compliance-SEC%20Rule%2017a--4(f)-blue)
![Security](https://img.shields.io/badge/Integrity-Merkle%20Proofs-green)
![License](https://img.shields.io/badge/License-MIT-lightgrey)

**Adamant** is a specialized storage engine designed for the financial sector's most rigorous requirement: **Immutability**.

In highly regulated environments (Banking, Trading, Insurance), logs and transaction histories must be stored in a **WORM (Write Once, Read Many)** format to prevent tampering. Adamant acts as a cryptographic middleware that guarantees data integrity not just through policy, but through mathematical proof.

> **Core Philosophy:** "Trust, but Verify." We trust the storage provider (AWS S3) to hold the bytes, but we verify the integrity of the history using our own cryptographic chain.

## 🏗️ Architectural Guarantees

Adamant is built in **Rust** to leverage its memory safety guarantees and prevent buffer overflows that could be exploited to bypass retention checks.

### 1. The WORM Enforcement Layer
Adamant sits between your application and the raw object storage.
* **Write Phase:** When a file is uploaded, Adamant calculates its SHA-256 hash and uploads it to S3 with **Object Lock** (Governance or Compliance mode) enabled.
* **Retention Logic:** The retention period is embedded in the object's metadata and signed. Attempts to overwrite or delete before expiry are rejected at both the API level and the storage layer.

### 2. Cryptographic Audit Trail (Merkle Tree)
To prove that no files have been silently deleted by a rogue admin with direct S3 access:
* Every upload is appended to a local **Merkle Tree**.
* The Root Hash of this tree is periodically anchored to a public ledger (e.g., Ethereum or a private audit log).
* **Audit:** An auditor can request a `Merkle Proof` to verify that a specific document exists in the set and has not been altered since creation.

### 3. Encryption at Rest & In Transit
* All data is encrypted using **AES-256-GCM** before leaving the memory buffer.
* Keys are managed via a KMS (Key Management System) abstraction, ensuring strictly separated duties.

## 🛠️ Tech Stack

* **Language:** Rust (Tokio for async I/O)
* **Storage Backend:** AWS S3 (with Object Lock enabled) / MinIO (for on-premise)
* **Integrity:** `rs-merkle` for tree construction.
* **API:** gRPC (Protobuf) for strictly typed ingestion.

## 🚀 Usage Example

### 1. Starting the Server
```bash
# Set up credentials and retention policy (e.g., 7 years)
export RETENTION_YEARS=7
cargo run --release
