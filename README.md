<p align="center">
  <img src="https://raw.githubusercontent.com/elementary/brand/main/community/community-black.png" alt="elementary community logo" width="128">
</p>

<h1 align="center">elementary OS WSL</h1>

<p align="center">
  <a href="https://github.com/invarianz/elementary-wsl/actions/workflows/build.yml">
    <img src="https://github.com/invarianz/elementary-wsl/actions/workflows/build.yml/badge.svg" alt="Build">
  </a>
</p>

A custom [WSL](https://learn.microsoft.com/en-us/windows/wsl/) distribution based on [elementary OS](https://elementary.io/) 8.

## Installation

Download `elementaryOS.wsl` from the [latest release](https://github.com/invarianz/elementary-wsl/releases/latest) and run (or just double click on the file):

```
wsl --install --from-file elementaryOS.wsl
```

## Features

- Systemd enabled by default
- Windows Terminal profile with io.elementary.terminal light/dark color schemes
- Start menu shortcut with elementary community icon
- First-run user setup (OOBE)
- **Terminal** and **AppCenter** available out of the box via WSLg

## How it's built

The build clones the upstream [`elementary/os`](https://github.com/elementary/os) repository and runs its live-build pipeline (`auto/config`, PPA sources, GPG keys, apt pinning, chroot hooks) unchanged. The same container environment (`debian:latest`, `--privileged`) is used.

The only differences from a vanilla elementary OS ISO build:

- **Lighter package list** — `elementary-minimal` + `elementary-standard` instead of `elementary-desktop` (no full Pantheon shell)
- **No kernel** — WSL provides its own (`--linux-packages none`)
- **Stops before ISO creation** — runs `lb bootstrap` + `lb chroot` only, then extracts the chroot as a rootfs tarball
- **GUI apps installed after hooks** — Terminal and AppCenter are added after the upstream blacklist hook runs, so they survive package removal
- **System-wide Flatpak remotes** — AppCenter and Flathub are configured system-wide so apps installed via AppCenter appear in the Windows Start Menu (WSLg only scans system-wide `.desktop` files)
- **WSL integration layer** — `wsl.conf`, OOBE script, systemd unit masking, Windows Terminal profile, and start menu icon
