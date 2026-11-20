# Black Moon OS — JalOS and the distro

[![License: MIT](https://img.shields.io/badge/license-MIT-blue)](LICENSE)

Two layers, one repo: **JalOS**, a complete teaching operating system written in Qasida, and **Black Moon OS**, a bootable Arch-based Linux distribution that ships JalOS as an app.

Status: this repo was last updated 29 Oct 2026. The JalOS copy here is the October 2026 version; the current copy, and the from-scratch kernel track it feeds, live in [MOON-OS](https://github.com/themythofsisyphusjayant/MOON-OS).

## JalOS — a complete teaching OS in Qasida

JalOS is every subsystem a modern OS has — bootloader, kernel, priority scheduler, virtual memory allocator, hierarchical filesystem with permissions, users, devices, networking, a package manager, system services, logging, and a ~30-command shell — implemented in [Qasida](https://pypi.org/project/qasida/) (formerly the Jal Pari Language; source in [RED-MOON-](https://github.com/themythofsisyphusjayant/RED-MOON-)) and running as a single program (`jalos.fg`).

| Subsystem | What JalOS does |
|---|---|
| Bootloader + kernel + init | Banner, RAM test, rootfs mount, daemon startup, login |
| Process management | Fixed process table, spawn, kill, ps/top |
| Scheduler | Priority scheduling in one-step quanta, tick log |
| Memory management | First-fit free-list allocator over 1024 simulated words, split + coalesce, `malloc`/`free`/`mem` |
| Filesystem | Hierarchical VFS (inodes by index), mkdir/write/cat/cp/mv/rm/ls, `/proc` pseudo-filesystem |
| Permissions + users | root/guest, file owners, owner-or-root checks, `su`, `useradd`, `/etc/passwd` |
| Devices | Console, loopback network device, disk (the VFS) |
| Networking | `ping` (loopback, simulated), `ifconfig`, `httpget` (real HTTP through the host stack) |
| Package manager | `pkg list/search/info/install/remove` — install really fetches the package source over HTTPS from the Qasida registry |
| Services + logging | cron (heartbeat every 5 ticks), syslog (`/var/log/syslog`), `logger`, `services` |
| Shell | ~30 commands, script files, single-command mode, `history`, `env`, `uptime`, `df`, `shutdown`, `ai`, `audit` |

## Run it

Install Qasida once (`pip install qasida`; current release 2.3.0). The scripts in this repo still default to the old `jalpari` command name — with the current release, set `FORGE` to `qasida`:

```sh
FORGE=qasida ./run.sh                       # the built-in demo tour
FORGE=qasida ./run.sh script.txt            # run a script file (see scripts/)
FORGE=qasida ./run.sh ps                    # a single command
```

`tests/check.sh` runs the demo tour and diffs it against `tests/expected_output.txt` — the tour is offline and deterministic, so it must match exactly.

## Black Moon OS — the distro

JalOS also ships inside a real, bootable Arch-based Linux distribution: XFCE desktop, Firefox/Chromium, GNOME Software, a guided installer, and Qasida. See [`distro/README.md`](distro/README.md) — build it with `./distro/build.sh` on Arch Linux, or via the GitHub Actions workflow (`.github/workflows/build-iso.yml`).

The distinction matters and is documented in the distro README: the distro is a real Linux system; JalOS itself runs *on top of* an OS and is not a kernel.

An ISO was built 27 Oct 2026 via GitHub Actions (workflow run succeeded). Live boot is untested.

## Limits

- **Not bootable — yet.** JalOS itself still runs on top of a host OS: it is a simulation, not a kernel. The language underneath it has since gained the pieces a bootable kernel needs — freestanding mode (`--freestanding`: no libc, own `_start`), raw memory (`peek32`/`poke32`), bitwise operators, CSR builtins, an `ecall` builtin, and a `trap_handler` hook. That kernel track now lives in [MOON-OS](https://github.com/themythofsisyphusjayant/MOON-OS) (Phases A–F done, idle task integrated, unified kernel). JalOS's ideas feed that kernel, but JalOS the program does not become it. What Qasida still lacks for kernel work: pointers and inline assembly.
- **No GUI.** No graphics builtins exist, so the desktop is the text shell.
- **No passwords worth trusting.** Passwords are compared as plain strings (no hashing builtin exists) — the user model demonstrates permissions, it does not secure anything real.
- **HTTPS works through proxies.** `pkg install` and `httpget` over HTTPS honour `https_proxy` / `HTTPS_PROXY` (plain HTTP proxy, `CONNECT` handshake before TLS; `no_proxy` bypasses it). Plain HTTP works everywhere regardless.
- **No persistence.** The VFS lives in memory; shutdown loses it.

## Layout

```text
jalos.fg            the teaching OS (single Qasida program)
run.sh              demo tour / script / single-command runner
scripts/            example script files
tests/              check.sh + expected_output.txt (exact-output regression check)
distro/             archiso profile, build.sh, distro README
INTEGRATION.md      how the two layers fit together
LICENSE             MIT
```

## Links

- The language: [RED-MOON- (Qasida source)](https://github.com/themythofsisyphusjayant/RED-MOON-) · [Qasida on PyPI](https://pypi.org/project/qasida/) (`pip install qasida`)
- The kernel track: [MOON-OS](https://github.com/themythofsisyphusjayant/MOON-OS)
- Distro details: [distro/README.md](distro/README.md)
