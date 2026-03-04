# Security Audit: PuppyRaffle Protocol 🦎

<div align="center">
  <img src="logo.png" alt="0xBreak Logo" width="250">
  <br>
  <strong>Lead Auditor:</strong> 0xBreak
  <br>
  <strong>Status:</strong> Completed (March 2026)
</div>

---

## 📌 About the Project
This repository contains a comprehensive security review of the **PuppyRaffle** smart contract. The project is a decentralized raffle system where users can enter to win unique puppy NFTs. 

The audit was conducted to identify potential vulnerabilities, optimize gas consumption, and ensure the safety of user funds.

## 🛡️ Audit Scope
- **Contract:** `PuppyRaffle.sol`
- **Language:** Solidity (^0.7.6)
- **Complexity:** Medium
- **Logic:** Raffle entry, winner selection, refund mechanisms, and NFT minting.

## 🚀 Vulnerabilities Discovered

| ID | Title | Severity | Status |
| :--- | :--- | :--- | :--- |
| **[H-01]** | Reentrancy Vulnerability in `refund` | 🔴 High | Reported |
| **[H-02]** | DoS via Unbounded Loop (O(n²)) in `enterRaffle` | 🔴 High | Reported |
| **[H-03]** | Weak Randomness (Predictable Winner) | 🔴 High | Reported |
| **[M-01]** | Integer Overflow in `totalFees` (uint64) | 🟠 Medium | Reported |
| **[M-02]** | Balance Poisoning (Fee Lock) | 🟠 Medium | Reported |

---

## 🛠 Tools Used
- **Foundry:** For writing Proof of Concept (PoC) exploits and fuzzing.
- **Obsidian:** For professional report generation.
- **Manual Review:** Deep analysis of the smart contract logic.

## 📂 Repository Structure
- `/audit-report.pdf`: The final, high-quality PDF report with detailed findings.
- `/test`: Foundry tests containing PoC exploits for discovered bugs.
- `/src`: Original (vulnerable) source code for educational purposes.

---

## 🧠 About 0xBreak
**0xBreak** is a security-focused initiative aimed at hardening Web3 protocols. We believe in the "Break to Build" philosophy — finding weaknesses before attackers do to ensure a safer DeFi ecosystem.

> **Breaking code to build a more secure future.**

## 📬 Contact
Ready for a security review? Reach out:
- **Twitter/X:** [@OxBreak]
- **Telegram:** [@OxBreak
- ]

---
