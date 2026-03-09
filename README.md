# Data Hop Firewall: Intelligent L7 Data Redaction for Envoy Proxy

**A high-performance WebAssembly (Wasm) filter written in Rust for Envoy Proxy, designed to prevent sensitive data leakage and enforce Zero-Trust data contracts at the network edge.**

---

## 🛡️ Motivation: The API Over-fetching Problem

Modern microservice architectures often suffer from **API Over-fetching** (CWE-201/CWE-209), where backends return complete database objects to the frontend, relying on the client to filter sensitive fields. This leads to accidental exposure of:

* Internal system paths and stack traces (Information Disclosure).
* Hashed passwords or internal service tokens.
* Sensitive PII (Personally Identifiable Information).

**Data Hop Firewall** moves the responsibility of data redaction from the application logic to the **Infrastructure Layer**, ensuring that sensitive data never leaves the internal network.

## 🚀 Technical Highlights

* **Engineered in Rust:** Leverages memory safety and zero-cost abstractions for high-performance L7 inspection.
* **WebAssembly (Wasm) Runtime:** Runs in a secure sandbox within Envoy, allowing for hot-reloading without proxy downtime.
* **Recursive JSON Redaction:** Deep-scans response payloads to identify and mask sensitive keys based on logical rules.
* **Fail-Closed Security:** Implements a strict security posture; if a payload cannot be safely parsed or inspected, the request is dropped to prevent potential leakage.

## 🏗️ Architecture

The filter intercepts HTTP responses at the Envoy Gateway. It monitors for specific triggers:

1. **Logical Data TTL:** Detects custom headers (e.g., `X-Data-TTL`) to initiate redaction.
2. **Global Masking:** Automatically masks common sensitive fields (passwords, secret keys) across all endpoints.
3. **Error Shielding:** Intercepts 5xx responses to prevent raw stack traces and server configuration details (like `web.config` snippets) from reaching the end-user.

## 🧪 Case Study: Server Error Masking

In recent observations of financial application gateways, it was identified that idle sessions combined with forced cache refreshes could trigger raw ASP.NET error pages. These pages expose internal server paths and configuration logic.

**Data Hop Firewall** mitigates this by identifying 5xx status codes and replacing the verbose system-generated HTML with a sanitized, generic JSON response, effectively hardening the application's surface area.

## 🛠️ Getting Started

### Prerequisites

* [Envoy Proxy](https://www.envoyproxy.io/) v1.28+
* [Rust](https://www.rust-lang.org/) with `wasm32-wasi` target.

### Build & Deploy

```bash
cargo build --target wasm32-wasi --release

```

Configure your `envoy.yaml` to include the compiled `.wasm` filter in the HTTP filter chain.
