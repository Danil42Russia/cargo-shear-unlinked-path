# cargo-shear `#[path]` unlinked file reproduction

Minimal reproduction for a cargo-shear `shear/unlinked_files` false positive involving a unit-test module loaded through a relative
`#[path]` attribute.

The project uses cargo-shear 1.13.4 and has no dependencies.

The unit-test module in [`src/lib.rs`](./src/lib.rs) loads [`tests/unit/smoke.rs`](./tests/unit/smoke.rs) through
`#[path = "../tests/unit/smoke.rs"]`. The empty [`build.rs`](./build.rs) makes cargo-shear scan the package root and discover the nested
test file.

## Reproduce

```sh
cargo install cargo-shear --version 1.13.4 --locked
cargo test --locked
cargo shear --locked --deny-warnings
```

`cargo test --locked` reports one passing test, `tests::linked_test`. However,
`cargo shear --locked --deny-warnings` exits with code 1 and reports:

```text
shear/unlinked_files

  ⚠ 1 unlinked file in `cargo-shear-path-repro`
  │ tests/unit/smoke.rs
  help: delete this file
```

## Expected behavior

cargo-shear should recognize `tests/unit/smoke.rs` as linked through the `#[path]` attribute in `src/lib.rs` and should not report
`shear/unlinked_files`.

## CI

The [GitHub Actions workflow](./.github/workflows/repro.yml) runs the reproduction on Ubuntu. The test passes, and the job is expected to
fail at the cargo-shear step with the false positive shown above.
