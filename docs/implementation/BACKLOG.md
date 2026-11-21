# Karkain Scientific Computing — First Implementation Backlog

## First vertical slice

**Slice goal.** Demonstrate real deterministic numerics end-to-end: read a fixed vector input, run a shape-checked dot product using the pure-Karkain `numerics` primitives (never raw float builtins), verify the known answer, and emit a byte-stable result file plus a one-line summary. This is the strongest candidate because `numerics` is PRESENT; the slice proves correctness and determinism, not GUI or GPU capability.

**Karkain modules/files expected.** `src/main.kark` (entry), `src/cli.kark` (file-driven invocation), `src/input_parser.kark`, `src/kernel.kark` (facade over `numerics`), `src/precision_contract.kark`, `src/serializer.kark`, `tests/self_test.kark`, `fixtures/vectors.txt`, `fixtures/expected-result.txt`. Stdlib: `numerics` (PRESENT - vectors, deterministic sqrt/exp, `assert` shape validation), `io` (PRESENT), `string`/`collections` (PRESENT), `json` (PARTIAL, optional), `crypto` (PRESENT, input digest). No `system()` call and no GUI.

**Exact input** (`fixtures/vectors.txt`, LF endings):
```
op=dot
a=1 2 3
b=4 5 6
```

**Exact expected observable output** (`out/result.txt`, byte-stable):
```
schema_version=1
operation=dot
result=32
precision=deterministic
```
Stdout on success: `dot result=32 (known-answer ok)`. Exit code 0. A second run must produce a byte-identical file. `a=1 2 4` with `b=4 5 6` must still yield a correct answer; mismatched lengths must be rejected by shape validation with a non-zero exit and no result file.

**Verification command pattern (Karkain v1.2.0, from the repository root):**
```
$env:KARKAIN_ENGINE = "go"
karkain lint src/main.kark
karkain run src/main.kark --target native-x86_64-windows
karkain build src/main.kark -o ksci --target native-x86_64-windows
.\ksci.exe
```
Go engine + C-free Windows target are the verified baseline; default `kcc` is environment-BLOCKED. `get_args()` returns an empty array, so the slice reads `fixtures/vectors.txt` directly.

## Backlog items

| ID | Feature | Files/modules | Dependencies | Acceptance criteria | Tests | Complexity | Blocking Karkain capability | Definition of done |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| SCI-001 | Vector input parser | `src/input_parser.kark` | none | Parses `op=`, `a=`, `b=` into ordered values | unit | S | none | Fixture parses to [1,2,3]/[4,5,6] |
| SCI-002 | Shape validation | `src/precision_contract.kark` | SCI-001 | Unequal lengths rejected with non-zero exit | unit, failure/recovery | S | none | Invalid fixture produces no result file |
| SCI-003 | Deterministic dot kernel | `src/kernel.kark` | SCI-002 | Uses `numerics`; result 32 for fixture | unit, known-answer | M | float builtins byte-unreliable -> primitives only | Result matches analytic answer |
| SCI-004 | Deterministic serializer | `src/serializer.kark` | SCI-003 | Byte-identical to expected result | unit, regression | S | none | Diff empty |
| SCI-005 | First vertical slice wiring | `src/main.kark`, `src/cli.kark` | SCI-001-004 | Compiled `.exe` writes result, prints summary | actual-compiled-executable | M | default `kcc` BLOCKED | `karkain build` then `.exe` exit 0 |
| SCI-006 | Precision/stability corpus | `tests/self_test.kark` | SCI-003 | Relative error budget recorded per primitive | unit, regression | M | none | `self-test` passes; budget documented |
| SCI-007 | Matrix operations | `src/kernel.kark` | SCI-003 | Matrix multiply/transpose shape-checked | unit, known-answer | L | none | Known-answer matrices pass |
| SCI-008 | Tensor shape-checked ops | `src/kernel.kark` | SCI-003, SCI-007 | Shape assertions on rank/size | unit | L | none | Invalid shape rejected |
| SCI-009 | JSON result export | `src/serializer.kark` | SCI-004 | Simple labelled fields only | unit, compatibility | M | `json` PARTIAL | Export fields verified |
| SCI-010 | Bench harness and text viz | `tests/bench.kark`, `src/text_viz.kark` | SCI-005 | Baseline recorded; ASCII/sparkline output | performance, E2E | L | GUI toolkit BLOCKED; `gpu` MODEL ONLY | Baseline documented, no GPU claim |

## Dependency order and gates

1. SCI-001 parser before SCI-002/003. 2. SCI-002 + SCI-003 before SCI-004/005. 3. Gate: known answer 32, byte-identical result, compiled `.exe` exit 0. 4. SCI-006 precision corpus before any precision claim. 5. SCI-007/008/009/010 expansion. No gate may pass using raw float builtins or a GPU-acceleration claim.

## Status

All items: **NOT STARTED**. No tests have been run for this application. Implementation must not begin until all designs and repository initialization are complete. GUI plotting BLOCKED; `gpu` is model-only with no vendor dependencies; `json` PARTIAL; Linux/macOS execution NOT VERIFIED.
