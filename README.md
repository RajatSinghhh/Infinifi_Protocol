# Infinifi Protocol Audit – Findings Summary

This document summarizes the key vulnerabilities discovered during the security audit of the protocol’s smart contract ecosystem. Each report is available in the same directory with a deeper technical breakdown, reproduction steps, and suggested remediations.

---

## 📑 Table of Contents

| ID | Finding Title                                                  | Severity<sup>†</sup> | Link                                               |
| -- | -------------------------------------------------------------- | -------------------- | -------------------------------------------------- |
| 1  | `multiVote` Function Can Be Exploited for Gas Griefing Attacks | Medium               | [Report 1](./finding-1-multivote-gas-griefing.md)  |
| 2  | Missing Return Value Checks for `approve()` Calls              | Medium               | [Report 2](./finding-2-missing-approve-checks.md)  |
| 3  | Oracle Price Fetch Should Handle Zero `assetPrice`             | Low                  | [Report 3](./finding-3-oracle-zero-price-check.md) |
| 4  | `mint()` May Fail Due to USDC Blacklist Restrictions           | Medium               | [Report 4](./finding-4-usdc-blacklist-risk.md)     |

> <sup>†</sup> *Severity levels are preliminary. Please refer to individual reports for detailed impact assessment.*

---

## 1 `multiVote` Function Can Be Exploited for Gas Griefing Attacks

* **Summary:** The `multiVote` function allows unbounded external contract calls with dynamic inputs, making it vulnerable to denial-of-service via gas exhaustion.
* **Risk:** Transaction failure, elevated gas costs, DoS.
* 👉 [Full report](./finding-1-multivote-gas-griefing.md)

## 2 Missing Return Value Checks for `approve()` Calls

* **Summary:** Multiple locations fail to verify the return value of ERC20 `approve()` calls, which can lead to unexpected behavior or approval failures.
* **Risk:** Loss of control over token allowances.
* 👉 [Full report](./finding-2-missing-approve-checks.md)

## 3 Oracle Price Fetch Should Handle Zero `assetPrice`

* **Summary:** The protocol does not check if `assetPrice` returned by the oracle is zero, which could lead to faulty logic or economic exploits.
* **Risk:** Protocol malfunction due to invalid price data.
* 👉 [Full report](./finding-3-oracle-zero-price-check.md)

## 4 `mint()` May Fail Due to USDC Blacklist Restrictions

* **Summary:** Circle’s blacklist enforcement may prevent `USDC` transfers during minting if the sender or protocol address is blacklisted.
* **Risk:** Mint transactions may fail unexpectedly, impacting usability and trust.
* 👉 [Full report](./finding-4-usdc-blacklist-risk.md)

---

### 📂 Directory Structure

```
.
├── finding-1-multivote-gas-griefing.md
├── finding-2-missing-approve-checks.md
├── finding-3-oracle-zero-price-check.md
├── finding-4-usdc-blacklist-risk.md
└── README.md   ← (you are here)
```

### ✍️ Contributing

If you encounter other edge cases or security risks worth disclosing, please open an issue or reach out securely.

---

© <?= date('Y') ?> <?= $ownerName ?? "AuditDrop" ?> – All rights reserved.
