# NIST Round 3 Additional Digital Signatures: PKI / X.509 Performance Evaluation

This repository benchmarks post-quantum digital-signature schemes in an OpenSSL/X.509 public-key infrastructure workflow rather than measuring only raw signature primitives.

The current work is a migration of the earlier NIST Additional Signatures Round 2 benchmark to the Round 3 candidate set and updated implementations.

> [!IMPORTANT]
> **Round 3 migration is still in progress.**
>
> The custom `liboqs` and `oqs-provider` integrations are under active build and validation. Some benchmark/test metadata in this repository still reflects Round 2 parameter sets and identifiers. Do not treat current benchmark output as final Round 3 results until the migration checklist is complete.

## NIST Round 3 scope

NIST announced nine Round 3 Additional Digital Signature candidates on May 14, 2026:

- FAEST
- HAWK
- MAYO
- MQOM
- QR-UOV
- SDitH
- SNOVA
- SQIsign
- UOV

HAWK has since been withdrawn from the NIST Additional Digital Signatures process. The active Round 3 benchmark therefore focuses on the remaining eight candidates. HAWK may be retained only as an explicitly labeled historical comparison.

Official NIST Round 3 page:
https://csrc.nist.gov/projects/pqc-dig-sig/round-3-additional-signatures

NIST Round 3 announcement:
https://csrc.nist.gov/News/2026/nist-advances-9-candidates-to-the-3rd-round-of-pqc

## Custom Open Quantum Safe forks

This project uses research forks of `liboqs` and `oqs-provider` so that Round 3 candidate implementations and controlled backend variants can be exposed through OpenSSL.

- `liboqs`: https://github.com/DoNotDisturbs/liboqs/tree/ds-0.16.0-round3-dev
- `oqs-provider`: https://github.com/DoNotDisturbs/oqs-provider/tree/0.10.0-round3-dev

The top-level repository also contains these repositories as Git submodules.

> [!WARNING]
> The submodule commit pointers on `pqc-benchmarking-v3/main` may temporarily lag the active Round 3 development branches while integration work is being validated. Update the submodule pointers only after the corresponding custom branch build has passed and the branch has been pushed.

## Current integration targets

| Family | NIST Round 3 status | Benchmark backend plan | Integration status |
|---|---|---|---|
| FAEST | Active | REF + AVX2 | Custom `liboqs` integration under validation |
| HAWK | Withdrawn | Historical only | Not part of the active Round 3 benchmark |
| MAYO | Active | Current liboqs Round 3 implementation | Present in current Round 3 stack |
| MQOM | Active | Current liboqs Round 3 implementation | Present in current Round 3 stack |
| QR-UOV | Active | REF + AVX2; AES/SHAKE variants | Custom `liboqs` integration under validation |
| SDitH | Active | REF + AVX2 | Custom per-variant Round 3 integration under validation |
| SNOVA | Active | Current liboqs Round 3 implementation | Present in current Round 3 stack |
| SQIsign | Active | REF + Broadwell | Custom `liboqs` integration under validation |
| UOV | Active | Current liboqs Round 3 implementation | Present in current Round 3 stack |

For custom candidate integrations, official candidate source files are kept unchanged. Adaptation is performed in the integration layer, such as CMake configuration, liboqs wrappers, namespace adapters, provider metadata, and OpenSSL-facing registration.

## Baselines

The benchmark can also include established or standardized signature schemes for comparison:

- ML-DSA / CRYSTALS-Dilithium
- SLH-DSA / SPHINCS+
- Falcon / FN-DSA work in progress at NIST
- RSA, when a classical baseline is useful

These baselines must be labeled separately from the NIST Additional Digital Signatures Round 3 candidates.

## What is benchmarked?

The main benchmark is the X.509 lifecycle for each enabled signature method:

1. private/public key-pair generation;
2. Certificate Signing Request (CSR) generation;
3. certificate signing / self-signed certificate generation;
4. certificate verification.

The benchmark records, where available:

- user and kernel CPU time;
- wall-clock time;
- CPU cycles;
- maximum resident set size / Peak RSS;
- public-key size;
- private-key size;
- signature size;
- encoded X.509 artifact sizes such as SPKI, CSR, certificate, and certificate-chain size.

The project is X.509-first. TLS measurements are a separate later stage and must not be mixed into the primary PKI benchmark unless explicitly enabled and documented.

## Repetition policy

The final benchmark protocol targets repeated measurements for stable summary statistics.

> [!CAUTION]
> During Round 3 integration testing, `run_benchmarks.py` currently defaults to **1 run** so failures can be debugged quickly. The final experimental run count must be changed and frozen before collecting publication-quality results. Previous documentation referring to 1000 runs should be treated as the target protocol, not the current script default.

For each final benchmark configuration, record the exact:

- repository commit;
- `liboqs` commit;
- `oqs-provider` commit;
- OpenSSL version;
- compiler and compiler version;
- CPU model and enabled instruction set;
- build flags;
- algorithm/backend identifier;
- run count.

## Build status

The previous `scripts/install_pqc_openssl.sh` workflow still contains Round 2 branch names and should **not** be used as the authoritative Round 3 setup procedure until it is rewritten and validated.

During the migration, build the three layers explicitly:

```text
custom liboqs
    ↓
custom oqs-provider generated against that liboqs source tree
    ↓
OpenSSL 3.x loading the resulting oqsprovider module
```

The current development branches are:

```text
liboqs:       ds-0.16.0-round3-dev
oqs-provider: 0.10.0-round3-dev
```

## Benchmarking

Once the Round 3 build is frozen and verified, benchmark execution remains centered on:

```bash
./run_benchmarks.py
```

Headless execution:

```bash
./run_benchmarking_headless.sh
```

Results are written under the project `results/` directory.

> [!TIP]
> Run performance experiments on an otherwise idle system. Record CPU frequency/governor settings and avoid mixing measurements collected under different build flags or hardware conditions.

## Testing

Family-level tests are dispatched with:

```bash
./tests/run_tests.sh <FAMILY>
```

For example:

```bash
./tests/run_tests.sh FAEST
./tests/run_tests.sh QR-UOV
./tests/run_tests.sh SDitH
./tests/run_tests.sh SQISign
```

The Round 2 test-family list is also being migrated. A family should be considered supported only when its current Round 3 provider identifiers successfully complete key generation, CSR creation, certificate creation, and verification.

## Reproducibility and source integrity

For research reproducibility:

- official candidate source files should remain byte-for-byte unchanged whenever the integration policy requires source preservation;
- build/integration wrappers should be clearly separated from official candidate source;
- REF and optimized backends should be independently addressable when both are benchmarked;
- frozen scientific artifacts must not be overwritten by later diagnostic runs;
- source/archive hashes and exact commit hashes should be recorded with final results;
- hard-coded Round 2 key/signature sizes must be removed or independently validated before Round 3 benchmarking.

## Cleaning

Project-level temporary benchmark output can be inspected or removed with:

```bash
./clean --dry-run
./clean --apply
```

Read the output before deleting submodule/build data.

## License

See [`LICENSE.md`](LICENSE.md).
