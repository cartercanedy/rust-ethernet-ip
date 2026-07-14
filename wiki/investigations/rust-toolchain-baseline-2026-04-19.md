# Rust Toolchain Baseline

## Summary

- `confirmed`: the workspace MSRV is Rust `1.88`.
- `confirmed`: all six workspace packages inherit the single
  `[workspace.package].rust-version` declaration in [Cargo.toml](../../Cargo.toml).
- `confirmed`: Rust `1.87.0` is incompatible with the locked dependency set,
  while Rust `1.88.0` compiles the complete workspace test suite with all
  features.
- `confirmed`: the repository remains on edition `2024`; lowering the MSRV did
  not change any Rust API, C ABI, or dependency.

## Current Understanding

- The MSRV is the oldest compiler that supports the complete Rust test suite,
  not the newest stable compiler used for development.
- Rust `1.88` is the current lower boundary because locked `time 0.3.47` and
  `time-core 0.1.8` each require Rust `1.88.0`.
- The root package, the four sibling crates under `crates/`, and the desktop app
  inherit the workspace MSRV. The standalone web backend is not a workspace
  member and does not inherit it.
- The dedicated CI job reads the MSRV from the root manifest, installs that
  exact toolchain, and runs the full workspace tests with all features.
- The earlier Rust `1.95` and `1.96` baselines reflected a current-stable
  development policy. They are superseded by the verified oldest-compatible
  policy for current mainline, but remain valid historical release evidence.

## Evidence

- Exact compiler search on `2026-07-14`, using
  `cargo +<version> test --workspace --all-features --locked --no-run`:
  - Rust `1.85.0`: failed because `time 0.3.47` and `time-core 0.1.8` require
    Rust `1.88.0`.
  - Rust `1.91.0`: passed.
  - Rust `1.88.0`: passed.
  - Rust `1.86.0`: failed on the same dependency requirements.
  - Rust `1.87.0`: failed on the same dependency requirements.
- The local stable upper bound, Rust `1.97.0`, also compiled the complete test
  suite with all features.
- `SKIP_PLC_TESTS=1 cargo +1.88.0 test --workspace --all-features --locked`
  passed on `2026-07-14`, including unit, integration, simulator, and doc tests.
- Stable Rust `1.97.0` passed workspace formatting and Clippy across all targets
  and all features with warnings denied.
- Manifest inheritance: [Cargo.toml](../../Cargo.toml),
  [crates/protocol/Cargo.toml](../../crates/protocol/Cargo.toml),
  [crates/tag-path/Cargo.toml](../../crates/tag-path/Cargo.toml),
  [crates/types/Cargo.toml](../../crates/types/Cargo.toml),
  [crates/udt/Cargo.toml](../../crates/udt/Cargo.toml), and
  [examples/desktop_app/Cargo.toml](../../examples/desktop_app/Cargo.toml).
- Enforcement: [.github/workflows/ci.yml](../../.github/workflows/ci.yml).
- Active user-facing policy: [README.md](../../README.md),
  [BUILD.md](../../BUILD.md), and [docs/API_STABILITY.md](../../docs/API_STABILITY.md).

## Open Questions

- `unclear`: a future dependency update could raise the exact boundary; CI and
  the binary-search procedure should be rerun when the locked dependency set
  changes materially.

## Related Pages

- [wrapper-parity/rust-vs-csharp.md](../wrapper-parity/rust-vs-csharp.md)
- [releases/0.8.0-validation-synthesis.md](../releases/0.8.0-validation-synthesis.md)
