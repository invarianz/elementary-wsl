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

Every divergence from a vanilla elementary OS ISO build is documented below.

### Package list

The desktop package list (`desktop.list.chroot_install`) is replaced with:

- `elementary-minimal`
- `elementary-standard`
- `wslu` (WSL utilities)

This replaces `elementary-desktop`, which pulls in the full Pantheon shell (compositor, dock, panel, greeter, etc.) — none of which are needed under WSL.

Two ISO-only package lists are also removed:

- `desktop.list.chroot_live` — packages for the live USB session
- `pool.list.binary` — packages bundled into the ISO's offline pool

### No kernel

The live-build option `--linux-packages` is changed from `linux-image` to `none`. WSL provides its own kernel; installing one inside the rootfs would be wasted space.

### Post-hook package installation

The following packages are installed **after** `lb chroot` finishes (i.e., after all upstream hooks have run):

- `io.elementary.terminal`
- `appcenter`
- `ubuntu-wsl`
- `flatpak`
- `elementary-default-settings`

The upstream live-build configuration includes a blacklist hook that removes packages not pulled in as dependencies of `elementary-desktop`. Since this build doesn't install `elementary-desktop`, these packages would be removed if they were in the normal package list. Installing them after the hooks ensures they survive.

`snapd` is purged immediately after installation. It gets pulled in as a dependency of `ubuntu-wsl`, but elementary OS uses Flatpak — not Snap — for app distribution.

### System-wide Flatpak remotes

Two system-wide Flatpak remotes are registered using `flatpak remote-add --system`:

- **AppCenter** — elementary's curated app catalog (`flatpak.elementary.io`)
- **Flathub** — the community Flatpak repository (`dl.flathub.org`)

Appstream metadata is pre-fetched at build time (`flatpak update --appstream`) so that AppCenter has a populated catalog on first launch.

The per-user Flatpak skeleton directory (`/etc/skel/.local/share/flatpak`) is also removed so that AppCenter defaults to system-wide installs.

This is required because WSLg only exposes **system-wide** `.desktop` files to the Windows Start Menu. Apps installed per-user would work inside the terminal but would be invisible in the Start Menu.

### Masked systemd units

29 systemd units are masked (symlinked to `/dev/null`) across six categories:

**Networking** — WSL manages DNS and the virtual network adapter:
`systemd-resolved.service`, `systemd-networkd.service`, `NetworkManager.service`, `avahi-daemon.service`, `avahi-daemon.socket`

**Tmpfiles and mounts** — WSL mounts its own `/tmp` and `/dev`:
`systemd-tmpfiles-setup.service`, `systemd-tmpfiles-setup-dev-early.service`, `systemd-tmpfiles-setup-dev.service`, `systemd-tmpfiles-clean.service`, `systemd-tmpfiles-clean.timer`, `tmp.mount`

**Hardware** — no physical hardware in WSL:
`ModemManager.service`, `wpa_supplicant.service`, `bolt.service`, `udisks2.service`, `iio-sensor-proxy.service`, `tpm-udev.path`, `fwupd-refresh.timer`, `secureboot-db.service`

**Cloud init** — WSL is not a cloud instance:
`cloud-config.service`, `cloud-final.service`, `cloud-init.service`, `cloud-init-local.service`, `cloud-init-hotplugd.socket`

**Crash reporting** — not useful in WSL:
`apport.service`, `apport-autoreport.path`, `apport-autoreport.timer`, `apport-forward.socket`

**Enterprise management** — not applicable:
`landscape-client.service`

### DNS

`/etc/resolv.conf` is deleted from the rootfs. WSL auto-generates this file on every boot using the Windows host's network configuration. Leaving a static copy would override WSL's DNS resolution.

### WSL configuration files

Three configuration files are copied into the rootfs:

- **`/etc/wsl.conf`** — enables systemd (`[boot] systemd=true`)
- **`/etc/wsl-distribution.conf`** — configures the OOBE (first-run user creation), distribution icon, and Windows Terminal profile path
- **`/etc/oobe.sh`** — the OOBE script itself, which runs on first launch to create the initial user account

### Windows integration

Two files are placed in `/usr/lib/wsl/` for Windows Terminal integration:

- **`terminal-profile.json`** — a terminal profile with elementary-themed light/dark color schemes
- **`elementary-community.ico`** — the elementary community icon, used in the Start Menu and Windows Terminal tab

### Output format

The build runs only `lb bootstrap` and `lb chroot`, then packages the chroot directory as a gzip-compressed tarball (`elementaryOS.wsl`). The `lb binary` stage (which assembles the ISO image, bootloader, and live filesystem) is skipped entirely. WSL imports distributions from rootfs tarballs, not ISOs.
