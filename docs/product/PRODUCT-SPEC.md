# Karkain Scientific Computing — Product Specification

**Document status:** Design COMPLETE. Research = COMPLETE. Repository creation = COMPLETE. Application implementation = NOT STARTED. Application compile/execute verification = NOT STARTED (toolchain path itself VERIFIED on Karkain v1.2.0).


## A. Product definition

**Product name.** Karkain Scientific Computing (`ksci`).

**Purpose and problem.** [VF] MATLAB is a commercial numeric-computation environment, [VF] GNU Octave is a mostly MATLAB-compatible high-level language, and [VF] Julia is an open, JIT-native scientific language. [RI] The gap is deterministic, auditable numerical results with zero vendor dependencies and a small, inspectable runtime. Karkain v1.2.0 has a real foundation: the `numerics` module is PRESENT with pure-Karkain vector/matrix/tensor operations, deterministic Newton/Taylor sqrt and exp, shape validation via `assert`, and no reliance on byte-unreliable float builtins for its primitives. This is the strongest systems-domain candidate. No application source exists yet, so all product features are planned.

**Vision.** A deterministic numerical kernel whose results are reproducible byte-for-byte on a given target, whose error behaviour is documented, and whose implementation is small enough to audit.

**Intended users/segments.** [RI] Researchers and engineers needing reproducible results; regulated/embedded users needing vendor-free auditable computation; educators wanting transparent implementations; Karkain platform evaluators.

**Usage scenarios.** (1) Run a vector/matrix operation from a text input file. (2) Verify results against analytic known-answer cases. (3) Benchmark a workload with `bench`. (4) Export results as text or simple JSON. (5) Render text-only tabular/sparkline visualisation.

**Market positioning.** A deterministic numerical kernel, not a MATLAB clone: no syntax compatibility claim, no GUI plotting, no vendor toolboxes.

**Primary value proposition.** Reproducible, shape-checked, vendor-free numerics with explicit precision, stability and performance contracts.

**Explicit non-goals/boundaries.** No GUI/plotting; no GPU-acceleration claim (`gpu` is a model with no vendor dependencies); no MATLAB syntax compatibility; no large package ecosystem; no language/`numerics`/compiler modification; no unvalidated float-builtin use where deterministic primitives exist.

## B. Functional architecture

| Module | Responsibility |
| --- | --- |
| `cli` | Invocation contract; constrained by `get_args()` returning an empty array (see F) |
| `input_parser` | Read deterministic text matrix/vector input from a fixed file |
| `kernel` | Thin, audited facade over `numerics` vector/matrix/tensor operations |
| `precision_contract` | Enforce documented error budgets and shape/validation assertions |
| `solver_examples` | Known-answer demonstrations (dot product, matrix multiply, sqrt/exp) |
| `serializer` | Deterministic text result output and simple JSON via PARTIAL `json` |
| `text_viz` | UnicodePlots-style tabular/sparkline output (no GUI) |
| `bench_harness` | Workloads compatible with `karkain bench`/`prof` |
| `diagnostics` | `self-test` for known-answer cases |

**Core workflows.** Parse input -> validate shapes -> execute kernel primitive -> check precision contract -> serialize result. **Admin workflows.** Select workload, set output format, run self-test. **Error/recovery workflows.** Shape mismatch or singular/ill-conditioned input yields a documented error code and no partial result; deterministic primitives are used exclusively. **Roles and permissions.** Single local user; no auth model. **Configuration/customization.** Versioned text config for precision mode and output format. **Import/export.** Text vectors/matrices in, text or simple JSON out; no external binary format assumed. **Integration requirements.** Stable exit codes and file output for scripts/CI.

## C. Competitor analysis

Competitors are **MATLAB**, **GNU Octave** and **Julia**, documented in `docs/competitor-analysis/COMPETITORS.md` and sourced from `domain5-systems.md` [all 2026-11-21].

| Capability (Karkain proposed) | MATLAB | Octave | Julia | Karkain v1.2.0 reality |
| --- | --- | --- | --- | --- |
| Deterministic math primitives | Not a stated goal | Not a stated goal | Not a stated goal | **YES** - `numerics` PRESENT [VF] |
| Vendor-free runtime | No (proprietary) | GPL deps | LLVM/Open deps | **YES** - pure Karkain |
| Native GUI/plotting | Yes | Yes | Package-based | **BLOCKED** - no GUI toolkit |
| GPU acceleration | Paid toolboxes | Limited | Yes (GPU.jl) | **MODEL ONLY** - no vendor deps |
| Large package ecosystem | Toolboxes | Forge packages | 12,000+ (site claim) | Small/local packages |
| Price | USD 1,050/yr or 2,625 listed | Free (GPL) | Free (MIT) | No product price defined |

**Platform/deployment.** [VF] Julia documents multiple platforms via LLVM; [UA] MATLAB/Octave platform matrices not individually verified; MATLAB uses licence manager, Octave distro/installers, Julia `Pkg` and binaries. **Licensing.** MATLAB "New Annual License" USD 1,050.00 and "New License" USD 2,625.00 listed from official store search snippets (direct fetch HTTP 403 on 2026-11-21; re-confirm before reliance); Octave GPL-3.0-or-later [VF]; Julia MIT [VF]. Julia's 12,000+ packages / 100M+ downloads are site claims, not independently audited [VF as claim]. **Strengths/limitations.** [RI] MATLAB: toolbox breadth but closed/costly; Octave: free/familiar but compatibility and graphics lag; Julia: open/fast but JIT latency and package maturity friction. **Market gaps.** [RI] Deterministic, auditable, vendor-free results. **Karkain differentiation.** PROPOSAL: lead with real `numerics` primitives and documented error/stability behaviour. **Features deliberately not copied.** MATLAB/Octave syntax, Simulink, toolboxes, JIT, large package registries, GUI plotting, GPU claim.

## D. Technical architecture

**The UI is CLI/TUI/service-first because Karkain v1.2.0 has no native GUI toolkit (BLOCKED); a GUI plotting layer is a planned layer contingent on ecosystem capability and is not in any current release.** Single-process CLI; no service in MVP. Boundaries follow B. UI architecture: text-only, deterministic column order. Domain model: `Vector`, `Matrix`, `Tensor`, `ComputationResult` with status and precision metadata. Storage: local files for input/results; no database required (E). API/interface: CLI subcommands, file I/O, exit codes. Event/message: none; `async` not needed. Logging/diagnostics: `self-test` known-answer report and optional run manifest. Configuration: versioned text config. Plugin/extension: planned Karkain-level module contributions, not binary plugins.

## E. Data design

**Entities.** `Vector` (length, ordered values), `Matrix` (rows, cols, row-major values), `Tensor` (shape, values), `ComputationResult` (operation, inputs digest, outputs, status, precision metadata), `ConfigFile`. **Relationships/ownership.** `ComputationResult` references input digests; local user owns files. **Validation.** Shapes validated via `assert` before operations (native `numerics` behaviour); singular/ill-conditioned detection documented. **Lifecycle.** parsed -> validated -> computed -> serialized. **Persistence.** File-based text input/result files; `db2` PRESENT/PARTIAL is not appropriate for immutable result artefacts; text keeps results diffable. **Transactions.** None required; single-file writes. Crash-atomic writes NOT VERIFIED (PARTIAL); readers tolerate partial files. **Backup/restore.** File copy. **Migration.** Version field on result format. **Import/export.** Plain text and simple JSON. **Audit history.** Input digest, operation and precision metadata recorded with each result.

## F. Karkain implementation architecture

Compiler/runtime: Karkain v1.2.0 only; baseline `KARKAIN_ENGINE=go` + `--target native-x86_64-windows`. Available stdlib: `numerics` PRESENT (real pure-Karkain vector/matrix/tensor, deterministic sqrt/exp, `assert` shape validation); `io` PRESENT; `math`/`string`/`random` PRESENT but unverified; `json` PARTIAL; `crypto`/`encoding` PRESENT (digests via sha256); `collections` PRESENT; `gpu` PRESENT as model only; `system`/`env`/`process`/`time` PARTIAL; `get_args()` returns an empty array in the checked-in module. Required ecosystem: none external. Missing dependencies: GUI toolkit (BLOCKED), full JSON DOM (PARTIAL), vendor GPU runtime (not present; `gpu` is model only). FFI/OS integration: none required for the numerical kernel; file I/O uses Karkain stdlib `io`. Native opportunities: all kernel work in pure Karkain on top of `numerics`. Build/packaging: one `.exe` via `karkain build`. **Blockers:** GUI/plotting BLOCKED; `gpu` is MODEL ONLY - no acceleration claim; `json` PARTIAL; float builtins byte-unreliable - all results must flow through deterministic `numerics` primitives; `get_args()` empty -> first slice reads a fixed input file; Linux/macOS execution NOT VERIFIED.

## G. Non-functional requirements

- **Precision.** Workload: known-answer cases (e.g. dot product, 4x4 matrix multiply, sqrt/exp at defined points). Method: compare `numerics` output to analytic reference values and record relative error. Criterion: maximum relative error below a budget established from baseline measurement and documented per primitive; byte-identical results on repeated runs of the same target. No invented fixed tolerance published before measurement.
- **Stability.** Workload: shape-mismatch, singular and ill-conditioned inputs. Method: `self-test` corpus. Criterion: shape violations caught by `assert`; ill-conditioned/singular cases return a documented status rather than silent garbage; no unvalidated float builtins in the result path.
- **Performance.** Workload: matrix multiply at sizes 10, 100, 400 on the ~4 GB machine. Method: `karkain bench` (and `prof` where available). Criterion: complexity consistent with expected O(n^3) dense multiply; thresholds set from measured baseline.
- **Memory.** Same workload; method: host RSS sampling; criterion: memory bounded by operand storage plus fixed overhead, no leak across repeated runs.
- **CPU/scalability.** Criterion: no busy loops; cost dominated by arithmetic; larger sizes degrade predictably.
- **Reliability/security/privacy.** Criteria: deterministic output; no untrusted input used as command strings; all data local; no network calls.
- **Portability/maintainability/accessibility/observability/recovery.** Text-only locale-independent output; module boundaries per B; precision metadata emitted with results; failed validation yields a documented error and no partial result.

## H. Compatibility and portability

Supported OS: Windows x86-64 VERIFIED toolchain; Linux cross-target output PRESENT, execution NOT VERIFIED; macOS NOT VERIFIED. Architecture: x86-64 first. Compatibility: Karkain v1.2.0 only. Package: single executable; install/uninstall = copy/delete. Config/data migration via version field; no backward compatibility before v1.2.0. Upgrade/rollback: keep prior binary and copy data. Cross-platform differences: deterministic primitives should be re-verified per target; no cross-target byte-identity claim until verified. Unsupported: GUI, GPU acceleration claims, syntax compatibility, non-x86-64 targets.

## I. Testing strategy

All tests PLANNED; none run for this application. Unit: known-answer vector/matrix/tensor cases (criterion: relative error within documented budget). Integration: parser -> kernel -> serializer (criterion: end-to-end correct result file). E2E: run from fixed input file (criterion: exit 0 and expected output). Platform-specific: Windows x86-64 first (criterion: compiled `.exe` runs). Security: input validation/injection tests (criterion: rejected). Performance: workload from G via `bench` (criterion: baseline recorded). Regression: known-answer corpus (criterion: no drift). Compatibility: v1.2.0 only. Actual-compiled-executable: `karkain build` then execute (criterion: exit 0, expected output). Failure/recovery: shape/singular cases (criterion: documented error, no partial result).

## J. Implementation roadmap

- **MVP / vertical slice:** deterministic vector/matrix operation from a fixed input file with shape validation and known-answer verification; gate = compiled executable produces byte-stable, correct result.
- **Core functional release:** expanded tensor ops, text serializer, `self-test` precision corpus; gate = precision/stability criteria from G met.
- **Advanced release:** bench harness, text visualisation, JSON export; gate = performance baseline documented and no regression.
- **Production-readiness:** docs, packaging, Linux/macOS verification; gate = planned suite executed and passing.
- **Dependencies:** kernel facade depends on `numerics`; precision contract depends on kernel; serializer depends on result model.
- **Risks/mitigations:** float builtin unreliability -> deterministic primitives only; `gpu` model-only -> no acceleration claim; GUI BLOCKED -> text viz; `get_args()` empty -> file input; no fictional dates or completion claims.
