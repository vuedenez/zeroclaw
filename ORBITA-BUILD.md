# Orbita build of ZeroClaw

This branch (`orbita/v0.7.5`) is the engine used by **OrbitaClaw**
(https://github.com/vuedenez/max_clawbot). It starts from upstream tag
`v0.7.5` (commit `a2988f0`) and carries a small number of Orbita-specific
patches on top.

**This is not an official ZeroClaw release.** The only official ZeroClaw
repository is https://github.com/zeroclaw-labs/zeroclaw (see `NOTICE`).
Binaries published from this fork are Orbita builds; do not report issues
with them upstream.

## Build contract

| Item | Value |
| --- | --- |
| Base | upstream tag `v0.7.5` (`a2988f0`), `Cargo.toml` version `0.7.5` unchanged |
| Branch | `orbita/v0.7.5` |
| Tags | `v<X.Y.Z>-orbita.<N>` — `X.Y.Z` is the upstream base and must equal `Cargo.toml` `version`; `N` is the Orbita build number, starting at 0 (no patches) |
| Workflow | `.github/workflows/orbita-release.yml` (derived from upstream `release-stable-manual.yml` at the same tag) |
| Trigger | push of a matching tag, or `workflow_dispatch` with an existing tag |
| Runner | `ubuntu-22.04` (glibc 2.35, upstream #3573) |
| Toolchain | Rust `1.93.0` (the toolchain of the upstream v0.7.5 release) |
| Targets | `x86_64-unknown-linux-gnu` (Orbita deploy hosts are amd64) and `aarch64-unknown-linux-gnu` (local arm64 image builds; cross-linked with `gcc-aarch64-linux-gnu` exactly as upstream) |
| Command | `cargo build --release --locked --features "channel-matrix,channel-lark,whatsapp-web" --target <target>` — default features are kept, the extra features are upstream's `RELEASE_CARGO_FEATURES` |
| Release profile | `Cargo.toml` as upstream (`panic = "abort"`, `lto`, `strip`) |
| Artifacts | `zeroclaw-<target>.tar.gz` per target, each containing `zeroclaw`, `LICENSE-APACHE`, `LICENSE-MIT`, `NOTICE`, `ORBITA-BUILD-INFO.txt` |
| Checksums | `SHA256SUMS` next to the tarballs: for each target, the tarball and the binary inside it |

Differences from the upstream release workflow, on purpose:

- no `web` job and no `web/dist` in the tarball — `embedded-web` is not in
  the release feature set, so the dashboard never reaches the binary;
- no desktop, docker, package-manager, website, tweet or discord jobs;
- `zeroclaw --version` still prints the upstream version; the Orbita marker
  lives in `ORBITA-BUILD-INFO.txt` and in the release notes.

Consumers pin the **binary's** SHA256 (the `zeroclaw (binary inside …)`
lines of `SHA256SUMS`, one per target) in their own Dockerfile. `SHA256SUMS` published next to the tarball is a
convenience, not a source of truth: anyone who can replace the tarball can
replace it too.

## Cutting a build

```bash
git checkout orbita/v0.7.5
# ... commit patches ...
git tag v0.7.5-orbita.<N>
git push fork orbita/v0.7.5 v0.7.5-orbita.<N>
```

The `Orbita Release` workflow builds, packages and publishes a GitHub
release for the tag. `Orbita CI` runs targeted tests on every push to the
branch (upstream `ci.yml` only runs on pull requests into `master`); its
manual run (`gh workflow run orbita-ci.yml --ref orbita/v0.7.5`) adds the
full workspace test suite, the same `cargo nextest run --workspace --exclude
zeroclaw-desktop` as upstream `ci.yml`.

## Patches

Listed in the release notes as `git log v0.7.5..<tag>`. Keep each patch a
single commit with a `feat(...)`/`fix(...)` subject so the list stays
readable.
