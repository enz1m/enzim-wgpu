# enzim-wgpu

The wgpu graphics stack maintained for enzUI and enzimOS.

An independently maintained fork of [gfx-rs/wgpu](https://github.com/gfx-rs/wgpu),
based on wgpu **29.0.4**. This repository preserves upstream Git history,
crate names, and the **MIT OR Apache-2.0** licensing. It is not an official
gfx-rs release.

## What this fork changes

- Reuse Vulkan command buffers without resetting their entire command pool.
- Cache Vulkan surface capabilities when the surface extent follows the swapchain,
  avoiding repeated Wayland compositor round trips during configuration.
- Expose Vulkan incremental-present damage hints so a compositor can repaint
  changed regions instead of the whole surface.
- Emit Naga SPIR-V composite layout decorations only for types reachable from
  memory interfaces.

See [FORK.md](FORK.md) for provenance, patch details, limitations, and the
[change log](CHANGELOG.md) for user-facing changes. The original
[upstream README](README.upstream.md) and documentation remain available.

## Repository layout

| Directory | Component |
| --- | --- |
| `wgpu/` | Public Rust graphics API |
| `wgpu-core/` | API validation and resource management |
| `wgpu-hal/` | Vulkan, Metal, Direct3D, and OpenGL backends |
| `naga/` | Shader translation and validation |
| `wgpu-types/`, `wgpu-naga-bridge/` | Shared types and shader integration |

The upstream workspace layout is retained. These components belong to one
repository and do not need separate clones.

## Use the fork

The canonical repository is [enz1m/enzim-wgpu](https://github.com/enz1m/enzim-wgpu).
This fork is distributed from Git; the Cargo package is still named `wgpu`.
There is no enzim-wgpu crates.io release.

```sh
git clone --branch enzim https://github.com/enz1m/enzim-wgpu.git
cd enzim-wgpu
cargo check -p wgpu --no-default-features --features std,vulkan,wgsl
```

For a direct Rust dependency, use the repository URL and pin a full commit:

```toml
[dependencies]
wgpu = { git = "https://github.com/enz1m/enzim-wgpu.git", rev = "<full commit SHA>", default-features = false, features = ["std", "vulkan", "wgsl"] }
```

Replace `<full commit SHA>` with the commit you reviewed. Existing renderers
that already depend on wgpu must use the same fork throughout their dependency
graph. A direct dependency alone does not replace their registry dependencies.
Using this fork requires updating the dependency graph of the application being built.

## Development

The upstream toolchain is Rust 1.93. For checks using an already installed
1.93.0 toolchain, use `cargo +1.93.0`.

Report fork-specific problems in this repository. For upstream documentation,
examples, and architecture, see the [upstream README](README.upstream.md).
Consult [FORK.md](FORK.md) before updating or rebasing the fork.

## License and attribution

MIT OR Apache-2.0, at your option. See [LICENSE.MIT](LICENSE.MIT) and
[LICENSE.APACHE](LICENSE.APACHE). Upstream copyright and attribution notices
are retained. enzUI modifications use the same dual license.
