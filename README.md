# 🛡️ Data Hop Firewall (Envoy Wasm Filter)

> **Zero-Trust Data Flow Control & L7 DLP for Microservices.**

### 🔬 Researcher's Note
> **Focus:** Vulnerability Research & Security Architecture.
>
> The core value of this project lies in the architectural research. The conceptualization, Zero-Trust security architecture, and identification of the attack vectors (React2Shell, CWE-209, Information Disclosure) are entirely my own design. Because my focus is vulnerability research and system internals rather than day-to-day Rust engineering, the actual Rust/WASM implementation was executed with AI assistance under my direct technical guidance. The true asset presented here is the identification of a structural flaw in a production environment and the engineering of an infrastructure-level mitigation.

---

## 📖 The Problem: API Over-fetching & CWE-209
In modern microservice architectures, **Data Bleed** and **API Over-fetching** are critical vulnerabilities. Backends often return massive database objects, relying on the frontend to filter sensitive fields. This inadvertently exposes sensitive PII, internal tokens, and stack traces to the network layer.

### 🚩 Case Study: Financial Gateway Error Shielding
During an analysis of a production banking environment, a specific sequence (idle session + forced refresh) caused the ASP.NET backend to crash. Instead of a generic error, it returned a 500 Internal Server Error exposing:
* Physical server directory paths.
* IIS and .NET versions.
* `web.config` manipulation suggestions.

This is a classic **CWE-209 (Information Exposure Through an Error Message)**.

## 🏗️ The Solution: "The Whisper Pipe" Architecture
I designed the **Data Hop Firewall** to shift sanitization responsibility from the application layer directly to the Envoy Proxy (Infrastructure Layer). 

Using a novel concept called **Logical Data TTL**, this Wasm filter actively enforces Data Contracts:
1. **Hop 0 (DB -> Backend):** Backend retrieves the full user object.
2. **Hop 1 (Backend -> Frontend):** Envoy Proxy intercepts the response.
3. **Interception & Redaction:** Recognizing the data boundary (via headers like `X-Data-TTL`), it surgically removes sensitive fields from the JSON payload in-memory or replaces raw stack traces with sanitized JSON.
4. **Delivery:** The Frontend receives a safe payload, completely blind to the original sensitive data.

## ✨ Core Capabilities
* **Engineered in Rust & Wasm:** Memory-safe, zero-cost abstractions, running in a secure sandbox within Envoy for hot-reloading.
* **Recursive JSON Redaction:** Deep-scans response payloads to identify and mask sensitive keys (`password`, `ssn`, `internal_token`).
* **Fail-Closed Security:** Operates on a strict zero-trust model. If JSON parsing fails or serialization panics, the payload is completely dropped to prevent leakage.

## 🛠️ Build Instructions

### Prerequisites
* [Rust Toolchain](https://rustup.rs/) (latest stable)
* Envoy Proxy (v1.28+)

### Compilation
Add the WebAssembly compilation target to your Rust environment:
(bash)
rustup target add wasm32-unknown-unknown
cargo build --target wasm32-unknown-unknown --release

The compiled binary will be generated at: target/wasm32-unknown-unknown/release/data_hop_firewall.wasm

Developed by Shay Mordechai | Vulnerability Researcher & Systems Architect
