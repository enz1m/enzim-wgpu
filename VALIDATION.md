# Import validation

Prepared on 2026-11-07 on Linux x86_64 with Rust 1.93.0.
This records source-import checks, not a complete upstream release qualification.

## Provenance and source checks

- The upstream clone is not shallow. The base commit retains 10,464 ancestral
  commits, including itself; upstream branch and tag history is retained.
- All nine 29.0.4 archive hashes match the enzUI provenance ledger.
- All seven imported Rust files match the current enzUI vendored files exactly
  after removing the added first-line modification notice.
- Upstream `LICENSE.MIT` and `LICENSE.APACHE` are byte-for-byte unchanged.
- The original upstream README is retained as `README.upstream.md`.
- Sibling dependencies use the upstream workspace paths, without local enzUI
  checkout paths or private dependencies added by this fork.

## Commands and outcomes

Build commands use `CARGO_INCREMENTAL=0 CARGO_PROFILE_DEV_DEBUG=0` to limit
scratch disk usage. The upstream lockfile is retained.

```sh
cargo +1.93.0 check --locked -p wgpu --no-default-features --features std,vulkan,wgsl
```

Passed. This checks the public API, core, HAL, Naga, bridge, and native Vulkan
platform dependency from this standalone checkout.

```sh
cargo +1.93.0 test --locked -p naga --features wgsl-in,spv-out \
  --test enzim_shader_layout shader_layout_excludes_composites_without_memory_interfaces
```

Passed: one test. The framework regression was adapted to depend directly on
Naga and use the upstream workspace's hashbrown collection types. It parses
and validates real WGSL, writes SPIR-V, and checks that local/private composites
omit explicit layouts while nested uniform, storage, and immediate-data layouts
retain their offsets, array strides, and matrix strides.

Negative control: temporarily restoring the upstream SPIR-V writer makes this
same test fail because a private variable reaches an explicitly laid-out type.
Restoring the imported writer makes it pass again.

```sh
cargo +1.93.0 clippy --locked -p wgpu-hal -p naga --tests \
  --features wgpu-hal/vulkan,naga/wgsl-in,naga/spv-out
cargo +1.93.0 fmt --all -- --check
git diff --check
```

Clippy completes with two `pattern_type_mismatch` warnings in the imported
SPIR-V patch, repeated for the library test target. No lint suppression or
functional changes were made to hide them. Formatting and whitespace checks pass.

## Not covered

The full WebGPU CTS, GPU runtime/validation-layer tests, cross-platform builds,
and historical compositor performance measurements were not run for this
source import. Existing enzUI applications remain on their current
vendored dependency graph; they were not repinned or rebuilt here.

## Signed history

All five commit signatures and the tag signature are verified locally. The
seven runtime Rust files and the shader-layout regression are byte-for-byte
identical to the validated source import. The upstream base, lockfile, licenses,
and preserved upstream README remain unchanged.
