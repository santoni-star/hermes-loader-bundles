# Hermes Loader bundles

Prebuilt layers for [Hermes Loader](https://github.com/santoni-star/hermes-loader),
so the phone does not have to run `apt-get install` and `uv pip install` inside
PRoot on first start.

This repository holds **only build output**: no source, no tokens, no user data.

| File | What it is |
|---|---|
| `manifest.json` | the entry point: bundle id, agent commit, Python version, and `sha256`/`bytes`/`unpacked` for every layer |
| release assets | `rootfs-<abi>.tar.xz` (Debian bookworm + apt packages) and `agent-<abi>.tar.xz` (`/opt/hermes`: checkout, `.python`, `.venv`, `bin/uv`) |

The app downloads `https://raw.githubusercontent.com/santoni-star/hermes-loader-bundles/main/manifest.json`,
then each layer from the release named by the manifest, and verifies every
archive against its SHA-256 before unpacking it.

The manifest is the only thing that has to be trusted. It is produced by
`tools/make-manifest.ps1` in the source repository, which also copies it to this
repository's `main` branch.

## Nothing here is required

A missing or broken bundle only makes the app log the reason and fall back to the
classic installer. Bundles are a speed-up, never a dependency.

## Layout of a bundle

* `rootfs` — Debian plus every package `setup.sh` installs, **without**
  `/opt/hermes`. Unpacked into `files/rootfs`.
* `agent` — `/opt/hermes` only, with a self-contained Python in
  `/opt/hermes/.python`. Unpacked into `storage/agent`.

Nothing is written to `storage/hermes` (the user's config, history and skills),
`storage/home`, `storage/db` or `storage/skills`.

Architectures: `arm64-v8a` (most phones), `x86_64` (emulators). The manifest
names the ABI, so a device only ever sees layers it can run.

## Rebuilding

In the source repository:

```bash
tools/bundles/build.sh                       # Docker, arm64-v8a
pwsh tools/make-manifest.ps1 -Dir out/bundles/arm64-v8a -Id 2026-10-28.1 -Publish
```

or run the `Bundle` workflow on GitHub (manual dispatch).
