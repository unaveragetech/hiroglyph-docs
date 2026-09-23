# Project Hiro vs Standard AES

## Executive Summary

Project Hiro should not be described as a cryptographic primitive that is stronger than standard AES.

At the primitive level:
- **standard AES**, especially AES-256-GCM, is more mature, more broadly reviewed, faster, and safer as a default choice for general-purpose encryption
- **Hiro** is a broader protection framework that combines encryption, symbolic obfuscation, system binding, file-type routing, and in-memory execution

That means Hiro can outperform a plain AES file-encryption workflow in a few operational areas:
- hiding that a file is even encrypted
- binding execution to a system profile
- executing protected payloads directly from memory
- routing protected non-Python content through addon handlers

But Hiro is weaker than standard AES in several critical areas:
- it is a custom design, so it has much less public review
- it is more complex, which increases attack surface
- its security properties depend on more moving parts than a standard AEAD construction
- its runtime model executes decrypted content, so the trust boundary is much larger than “decrypt bytes safely”

The right framing is:

> Hiro is a packaging and protected-execution system built around custom cryptographic and runtime layers. It may outperform plain AES workflows in concealment and execution control, but it does not replace AES-256-GCM as the standard benchmark for cryptographic assurance.

## Comparison Matrix

| Dimension | Standard AES-256-GCM | Project Hiro |
|---|---|---|
| Primitive maturity | Excellent | Limited compared to AES |
| Public review | Extensive | Narrow |
| Interoperability | Excellent | Custom |
| Authenticated encryption | Native AEAD | Integrity wrapper, but not a standard AEAD mode |
| Performance | Typically faster | Slower due to hashing, symbolic encoding, and wrapper logic |
| Detectability on disk | Ciphertext is visibly encrypted | `.glyph` output is visually disguised |
| System binding | Not built in | Built in |
| In-memory protected execution | Not built in | Built in for Python, addon-dependent for other types |
| Multi-language execution | External concern | Integrated through addons |
| Implementation complexity | Lower | Higher |
| Risk of regression | Lower in standard libs | Higher due to custom runtime and packaging paths |

## Where Hiro Outshines a Plain AES Workflow

### 1. Concealment and presentation

A normal AES-encrypted blob still looks like encrypted data. Hiro's symbolic glyph output makes static recognition harder and gives it plausible deniability at the storage layer.

### 2. System-bound distribution

Hiro can derive keys from a machine fingerprint and export/import those variables for controlled execution. Plain AES does not provide that behavior by itself.

### 3. Protected execution workflow

AES is usually just a decryption primitive. Hiro includes:
- encode
- decode
- runtime execution
- addon dispatch
- no-temp execution paths

That is a meaningful product advantage when the goal is controlled code delivery rather than generic encryption.

### 4. File-type aware runtime

Hiro can package Python, HTML, JavaScript, Java, TypeScript, and batch payloads behind one encoded format and route them through one runner surface.

## Where Standard AES Still Wins

### 1. Cryptographic confidence

AES-256-GCM is standardized, heavily reviewed, and widely deployed. A custom stream-cipher-style design does not have the same level of assurance.

### 2. Simpler security story

AES-256-GCM already gives confidentiality and integrity in one standard construction. Hiro combines:
- custom key derivation rules
- custom symbolic encoding
- custom runtime execution
- addon loading

Each extra layer is useful, but each one also increases the number of ways the system can fail.

### 3. Ecosystem support

AES integrates cleanly with standard tooling, hardware acceleration, HSMs, compliance programs, and cross-language stacks.

### 4. Performance

Hiro does more work:
- hashing and key stretching
- symbolic conversion
- optional error markers
- runtime routing

So it should be expected to be slower than direct AES file encryption.

## Can Hiro Be “Broken”?

No serious system should be described as impossible to break. The right question is which attack classes it resists well, which ones it only partially addresses, and where its main exposure lives.

## Attack Surface Review

### What current tests already support well

The repository already contains strong lower-level coverage for:
- key derivation determinism and separation
- encryption/decryption round trips
- integrity checks
- glyph encoding round trips
- addon behavior
- some security benchmarking and brute-force analysis

Relevant files:
- [tests/test_hiro_comprehensive.py](C:/Users/b0052/Desktop/python%20projects/Projects/hiroglyph/tests/test_hiro_comprehensive.py)
- [tests/test_addon_memory.py](C:/Users/b0052/Desktop/python%20projects/Projects/hiroglyph/tests/test_addon_memory.py)
- [tests/test_rainbow_table.py](C:/Users/b0052/Desktop/python%20projects/Projects/hiroglyph/tests/test_rainbow_table.py)
- [docs/HIRO_SECURITY_WHITEPAPER.md](C:/Users/b0052/Desktop/python%20projects/Projects/hiroglyph/docs/HIRO_SECURITY_WHITEPAPER.md)

### Where the biggest residual risks are

#### 1. Custom cryptographic construction risk

Even if the implementation is clean, a custom scheme is harder to trust than AES-256-GCM because it has less external scrutiny.

#### 2. Runtime execution risk

If a validly encrypted payload is malicious, Hiro will still execute it. The system protects distribution and packaging, not payload benevolence.

#### 3. Addon boundary risk

Non-Python payloads rely on external runtimes and addon handlers. That expands the trust boundary to:
- Node.js
- browsers
- Java tools
- Windows command processor

#### 4. Build/packaging drift

A working Python source flow can still fail as a packaged exe if bundled resources, imports, or addon dependencies change.

## Recommended Security Positioning

The strongest accurate claims are:

- Hiro provides a protected execution framework with symbolic obfuscation and system binding.
- Hiro can make static identification and casual inspection harder than a plain AES-encrypted file workflow.
- Hiro supports in-memory execution paths that reduce plaintext file artifacts for supported runtimes.

Claims to avoid:

- “Hiro is stronger than AES”
- “Hiro cannot be broken”
- “Hiro replaces standard AEAD best practices”

## Exe Regression Strategy

The packaged executable is now covered by:

- [tests/test_exe_regression.py](C:/Users/b0052/Desktop/python%20projects/Projects/hiroglyph/tests/test_exe_regression.py)
- [exe_assets/local_import_script.py](C:/Users/b0052/Desktop/python%20projects/Projects/hiroglyph/exe_assets/local_import_script.py)
- [exe_assets/async_workload.py](C:/Users/b0052/Desktop/python%20projects/Projects/hiroglyph/exe_assets/async_workload.py)
- [exe_assets/dashboard.html](C:/Users/b0052/Desktop/python%20projects/Projects/hiroglyph/exe_assets/dashboard.html)
- [exe_assets/analytics.js](C:/Users/b0052/Desktop/python%20projects/Projects/hiroglyph/exe_assets/analytics.js)

## What the exe suite checks

- help and command surface
- addon discovery
- password encode/decode roundtrip
- system export and system-key decode flow
- AES encode/run for a Python script that imports local modules
- AES encode/run for a long-running async Python script
- HTML addon path through the exe
- JavaScript addon path through the exe
- wrong-key failure behavior
- timing output and file-size sanity checks

## Recommended Ongoing Test Layers

### Layer 1: Cryptographic and algorithm tests

Keep running:
- `tests/test_hiro_comprehensive.py`
- `tests/test_crypto_security.py`
- `tests/test_rainbow_table.py`

These protect the lower-level behavior.

### Layer 2: Addon tests

Keep running:
- `tests/test_addon_memory.py`

This protects execution backends and cleanup behavior.

### Layer 3: Packaged product tests

Run:
- `tests/test_exe_regression.py`

This protects the shipped executable.

### Layer 4: Manual release smoke

Before release:
1. rebuild the exe
2. run addon listing
3. run a Python glyph
4. run an async glyph
5. run an HTML glyph in `--no-temp`
6. run a JavaScript glyph

## Suggested Hardening Next

If the goal is stronger assurance over time, the next improvements should be:

1. move the cryptographic design closer to a standard AEAD model
2. add tamper and corruption corpus tests for the exe surface
3. add a machine-readable benchmark report artifact for release builds
4. add CI that builds the exe and runs `tests/test_exe_regression.py`
5. consider a signed release process for the packaged executable
