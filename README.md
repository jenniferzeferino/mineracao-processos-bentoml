# Process Mining in Authentication Logs: Concurrency & Security Analytics

![Python](https://img.shields.io/badge/Python-3.10%2B-blue)
![Domain](https://img.shields.io/badge/Domain-Cybersecurity%20%26%20Process%20Mining-orange)
![License](https://img.shields.io/badge/License-MIT-green)

## Overviews & Business Context
In modern cloud architectures and identity management systems (IAM), monitoring authentication workflows is critical for both user experience and security. While traditional Market Basket Analysis evaluates event co-occurrence (un-ordered sets), **Process Mining** models execution traces to ensure system events strictly adhere to designed state-machine flows.

This project implements process discovery and sequential association rule mining on an event log of **2,000 authentication sessions**. The objective is to evaluate multi-factor authentication (MFA) funnel conversion and detect security anomalies caused by out-of-order execution or race conditions.

---

## Dataset Architecture
The raw data consists of structured audit logs (`JSONL` format). Each log entry represents a discrete telemetry event with three primary attributes:

* `session_id`: Unique identifier of the authentication attempt (Case ID).
* `evento`: Activity name (`falha_senha`, `senha_aceita`, `desafio_mfa`, `falha_mfa`, `login_sucesso`).
* `timestamp`: ISO-8601 timestamp used to sequence events chronologically.

---

## Expected State Machine (Happy Path)
The baseline authentication workflow requires password verification prior to secondary factor challenges:

1. **Password Verification:** Zero or more `falha_senha` followed by `senha_aceita`.
2. **MFA Challenge (Optional):** `desafio_mfa` is triggered **only after** `senha_aceita`.
3. **Completion:**
   * **Standard Flow:** `senha_aceita → login_sucesso`
   * **MFA Flow:** `senha_aceita → desafio_mfa → login_sucesso`

---

## Key Metrics & Formulations

### Sequential Rule Association
* **Support:** Proportion of total sessions containing the complete sequence:
  $$\text{Support} = \frac{\text{Sessions matching } (\text{LHS} \rightarrow \text{RHS})}{\text{Total Sessions (2000)}}$$

* **Confidence:** Conditional probability of completing the workflow given the prefix execution:
  $$\text{Confidence} = \frac{\text{Sessions matching } (\text{LHS} \rightarrow \text{RHS})}{\text{Sessions matching LHS}}$$

---

## Analysis & Analytical Findings

### 1. Execution Trace vs. Itemset Analysis
For session `descoberta-0001`:
* **Trace (Ordered Sequence):** `falha_senha → falha_senha → senha_aceita → desafio_mfa → login_sucesso`
* **Itemset (Unordered Set):** `{desafio_mfa, falha_senha, login_sucesso, senha_aceita}`

*Key Takeaway:* While itemsets capture co-occurrence, traces reveal operational loops (e.g., multiple failed password attempts) and causal dependency.

---

### 2. MFA Conversion & Retention
Evaluated rule: `(senha_aceita → desafio_mfa) → login_sucesso`

* **Prefix Match Count:** `[Y]` sessions
* **Full Rule Match Count:** `[X]` sessions
* **Support:** `[X]/2000`
* **Confidence:** `[X]/[Y]`

**Engineering Insights:**
* **Support:** Quantifies the global volume of users successfully completing two-factor authentication.
* **Confidence:** Demonstrates low friction in the MFA challenge stage, measuring user completion rate once secondary verification is prompted.

---

### 3. Security Anomaly Detection: Out-of-Order Execution

* **Pattern A (Co-occurrence Set `{senha_aceita, desafio_mfa}`):** Support = `[Z]/2000`
* **Pattern B (Anomalous Trace `desafio_mfa → senha_aceita`):** `[N]` instances
