# Envoy Dynamic Modules Rust SDK Crate Universe Example

This is a minimal Bazel + `rules_rust` Crate Universe example that depends on
Envoy's dynamic modules Rust SDK using the Git dependency style documented by
the SDK:

```toml
[dependencies]
envoy-proxy-dynamic-modules-rust-sdk = { git = "https://github.com/envoyproxy/envoy", tag = "v1.38.2" }
```

Crate Universe ingests that `Cargo.toml` through `crate.from_cargo`.

The Envoy SDK crate lives under:

```text
source/extensions/dynamic_modules/sdk/rust
```

Its build script currently reads the ABI header via a parent-directory relative
path:

```text
../../abi/abi.h
```

That path works when building inside the full Envoy repository, where the
canonical header exists at:

```text
source/extensions/dynamic_modules/abi/abi.h
```

However, Crate Universe generates a Bazel external repository rooted at the Rust
SDK crate directory. In that generated repository, the surrounding Envoy source
tree is not present, so `../../abi/abi.h` cannot be resolved by the SDK build
script.

## Reproduce

```sh
bazel build //:example
```

Expected result: the Envoy SDK build script fails when `bindgen` tries to read
`../../abi/abi.h`.

Observed failure:

```text
cargo:rerun-if-changed=../../abi/abi.h

thread 'main' panicked at .../build.rs:27:6:
Unable to generate bindings: NotExist("../../abi/abi.h")
```
