# Envoy Dynamic Modules Rust SDK Example

This is a minimal Cargo + Bazel `rules_rust` Crate Universe example that depends
on Envoy's dynamic modules Rust SDK as a Git dependency.

```toml
[dependencies]
envoy-proxy-dynamic-modules-rust-sdk = { git = "https://github.com/dshnkao/envoy", rev = "a45c65d075d26b976c96a302a06fc52e7bc45960" }
```

The pinned Envoy commit contains the Rust SDK ABI header packaging fix. The SDK
crate now exposes the ABI header at:

```text
source/extensions/dynamic_modules/sdk/rust/abi/abi.h
```

That file is a symlink to Envoy's canonical ABI header:

```text
source/extensions/dynamic_modules/abi/abi.h
```

The SDK build script reads `CARGO_MANIFEST_DIR/abi/abi.h`, so both Cargo and
Crate Universe can build the Git dependency from the crate directory while still
using the canonical Envoy ABI header.

## Verify with Cargo

```sh
LIBCLANG_PATH=/lib/x86_64-linux-gnu cargo build
```

This exercises Cargo's Git dependency checkout directly. If your system already
has libclang discoverable by `clang-sys`, plain `cargo build` is sufficient.

## Verify with Bazel

```sh
bazel build //:example
```

This exercises `rules_rust` Crate Universe, which ingests `Cargo.toml` through
`crate.from_cargo` and builds the Envoy SDK as an external Rust crate.

## Previous failure

Before the Envoy patch, the SDK build script read the header via:

```text
../../abi/abi.h
```

That path works inside the full Envoy source tree, but it fails once Cargo or
Crate Universe builds the SDK from the Rust crate directory. The failure looked
like:

```text
cargo:rerun-if-changed=../../abi/abi.h

thread 'main' panicked at .../build.rs:27:6:
Unable to generate bindings: NotExist("../../abi/abi.h")
```
