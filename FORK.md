# Fork provenance and maintenance

## Upstream

- Project: [gfx-rs/wgpu](https://github.com/gfx-rs/wgpu).
- Base: `e99f5305ded96ff7006f0714d043a7f735bd45c2` (wgpu 29.0.4).
- Repository: [enz1m/enzim-wgpu](https://github.com/enz1m/enzim-wgpu).
- Development branch: `enzim`.
- Fork author/committer: Bernard (`enz1m`).
- License: MIT OR Apache-2.0, including enzUI modifications.
- Original upstream README: [README.upstream.md](README.upstream.md).

The `enzim` branch starts at the exact upstream commit recorded in the nine
crate archives consumed by enzUI. Upstream ancestry is retained unchanged.
Each of the four fork patches is recorded in an independent, signed commit.

## Imported changes

### Vulkan command-buffer reuse

`wgpu-hal/src/vulkan/device.rs` creates command pools with
`RESET_COMMAND_BUFFER`. `command.rs` lets `vkBeginCommandBuffer` reset reused
buffers implicitly instead of resetting every buffer in a pool. Discarded
buffers that are still recording are reset explicitly before reuse.

### Surface capability caching

`wgpu-hal/src/vulkan/swapchain/native.rs` caches capabilities for an adapter
while the surface reports no fixed current extent, as on Wayland. Fixed-extent
surfaces continue querying on configuration. The cache does not track later
format changes reported through compositor dmabuf feedback. That limitation
needs review before treating this policy as generally upstreamable.

### Incremental present

The Vulkan backend requests `VK_KHR_incremental_present` where supported and
exports `wgpu_hal::vulkan::set_present_damage`. Rectangles are consumed by the
next present on the calling thread. `None` means the full image; an empty list
becomes a one-pixel region because Vulkan treats zero rectangles as full damage.
The hint is discarded when the extension is unavailable. Each image must still
contain the full frame. This is a fork-specific, thread-local HAL API.

### SPIR-V memory-interface layouts

`naga/src/back/spv/writer.rs` emits array strides and structure member offsets
for types reachable from uniform, storage, or immediate-data globals. Purely
function/private composites omit those decorations. Nested memory-interface
layouts and existing std140 wrappers retain their explicit decorations.
Types shared between memory interfaces and function variables are not split
into separate SPIR-V types by this patch.

## Import audit

All nine original crates.io archives were downloaded and SHA-256 checked
against the enzUI provenance ledger before import. The complete vendored
files were compared to those archives. The differences were seven Rust files
and sibling dependency paths in eight generated Cargo manifests; no extra
source files were present. The seven Rust files were transferred to their
original upstream locations, with a modification notice added to each file.
The existing framework shader-layout regression was also adapted into
`naga/tests/enzim_shader_layout.rs` so it can run in this standalone repository.

The upstream Cargo workspace already supplies sibling paths, so generated
archive manifests were not copied over the upstream manifests. This retains
the complete workspace, including examples, tests, and unpublished tooling.
`wgpu-types` is unchanged upstream source in this workspace; enzUI's vendored
snapshot previously used the registry release of that same package.

| Archive (29.0.4) | SHA-256 |
| --- | --- |
| `naga` | `b2bf919621e7975acb27d881bae2fb993e0d45c8e0446e85e6272971e00dc8df` |
| `wgpu` | `76e8840e1ba2881d4cbb18d2147627a56af426ff064c0401eb0c8410c6325d07` |
| `wgpu-core` | `2f519832254e56965a9940c4af57dcb75f702b6f6fa4a0b172f685395843a4d7` |
| `wgpu-core-deps-apple` | `f5e39e26c4c0e07589e67d18546cf79ff45383659fc72fca4dd293358a0347f3` |
| `wgpu-core-deps-emscripten` | `01e09be551dc939498bdd5f6b2c66e55ab275dad25825267a08605a80fc9f0af` |
| `wgpu-core-deps-wasm` | `af1fb1798be2a912497d4c224f72d39bb0cb34af50e8bcc29865bc339c943059` |
| `wgpu-core-deps-windows-linux-android` | `4e592c1bbef6ad047647ae6e666ebd8cee7a32bb4544d9700ec96cbf73230257` |
| `wgpu-hal` | `97ace1c17727311c22a46e4e3faf56ea6de81af99dcc839bdfb54857b94d448d` |
| `wgpu-naga-bridge` | `95226013f547544b223281cd16a4fb549aa9dcb562adbda0faae4c73ffbbc161` |

## Updating the fork

Keep upstream history intact and import upstream changes onto `enzim`. Review
each patch against the new upstream base and drop patches superseded upstream.
Update this provenance record and the changelog. Pin downstream dependencies
to a reviewed commit and check that they resolve a single compatible wgpu and
Naga stack. Consumers must explicitly update their dependency pins to adopt a new revision.

## Distribution and validation

This fork is distributed from Git and is not published to crates.io.
Verification performed for this import is recorded in
[VALIDATION.md](VALIDATION.md). Historical runtime measurements from the enzUI
ledger were not rerun as part of repository preparation.
