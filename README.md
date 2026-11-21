# Karkain Scientific Computing

A planned deterministic numerical kernel for vectors, matrices and tensors, built exclusively on the pure-Karkain `numerics` module in Karkain v1.2.0.

## Status

| Area | Status |
| --- | --- |
| Research | COMPLETE |
| Product design | COMPLETE |
| Repository creation | COMPLETE |
| Implementation | NOT STARTED |
| Compile/execute verification of the application | NOT STARTED (toolchain itself VERIFIED on Karkain v1.2.0) |

## What it does

Karkain Scientific Computing is intended to expose a small, auditable API for dense linear algebra and deterministic elementary math. Its foundation is the `numerics` module, which is PRESENT in Karkain v1.2.0 and provides pure-Karkain vector, matrix and tensor operations with deterministic Newton/Taylor implementations of sqrt and exp, shape validation via `assert`, and no reliance on byte-unreliable float builtins for its primitives. The product will add a CLI/TUI front end, reproducible input/output formats, and explicit precision, stability and performance contracts. This is the strongest systems-domain candidate because the core capability already exists in the language; however, no application source has been written yet, so every feature below is planned, not implemented. Results will be positioned as deterministic and vendor-free, not as a clone of any commercial numerical environment.

## Who it serves

[RI] Researchers and engineers who need reproducible numerical results with a small inspectable runtime; regulated or embedded-context users who require auditable, vendor-free computation; and educators who want transparent implementations rather than opaque vendor libraries. It is not aimed at users needing GUI plotting or a large third-party package ecosystem today (both BLOCKED/limited).

## Planned features

- Core numerics (planned): vector add/scale/dot (planned); matrix multiply/transpose (planned); tensor shape-checked operations (planned); deterministic sqrt/exp primitives (planned).
- Precision and stability (planned): documented error budgets versus analytic references (planned); shape/validation assertions (planned); defined behaviour for singular and ill-conditioned input (planned).
- I/O (planned): text matrix/vector input (planned); deterministic result output (planned); JSON result export via the PARTIAL `json` module (planned).
- Tooling (planned): `bench`-compatible performance harness (planned); self-test corpus of known-answer cases (planned).
- Text visualisation (planned): Unicode/ASCII UnicodePlots-style tabular and sparkline output (planned).
- Platform (planned): Windows x86-64 first (planned); Linux/macOS after execution verification (planned).

## Explicit non-goals

- No GUI plotting or interactive notebook (no native GUI toolkit; BLOCKED).
- No GPU-acceleration claim: the `gpu` module is a model with no vendor dependencies.
- No MATLAB syntax compatibility claim; import/export shims only.
- No claim of a large package ecosystem or vendor toolboxes.
- No modification of the Karkain language, `numerics` module or compiler.
- No unvalidated use of float builtins where `numerics` deterministic primitives exist.

## Karkain foundation

Exclusive foundation: Karkain Language v1.2.0. All application source will be Karkain. No other implementation language. No compiler modifications.

## Platform support status

Windows x86-64 toolchain VERIFIED (C-free native target with the Go front-end). Linux cross-target output PRESENT but execution NOT VERIFIED on this host. macOS NOT VERIFIED. Numerical behaviour itself is NOT VERIFIED for this application because no application-level smoke test has been run.

## Build and run status

No application binary exists yet. The intended generic commands are `karkain build src/main.kark -o ksci` and `karkain run src/main.kark`; these become active when implementation starts. The verified toolchain baseline on this machine uses `KARKAIN_ENGINE=go` with `--target native-x86_64-windows`, which compiles and runs a real executable. Application-level compile/run verification is still NOT STARTED.

## Known blockers

- Native GUI toolkit/plotting: BLOCKED - text output only.
- `gpu` module: model/abstract executor with no vendor dependencies - no acceleration claim.
- `json` module: PARTIAL - suitable for simple field extraction, not full DOM/streaming.
- Float builtins documented byte-unreliable; results must come from `numerics` deterministic primitives.
- Default `kcc` engine: BLOCKED in this environment; use the Go engine + C-free native target.
- Linux/macOS execution: NOT VERIFIED.

## Repository

Source: https://github.com/ajit-ai/Karkain-Scientific-Computing
Issues: https://github.com/ajit-ai/Karkain-Scientific-Computing/issues
License: Apache-2.0.
