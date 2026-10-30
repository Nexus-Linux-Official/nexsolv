# nexsolv

Arch Linux adaptation of [libsolv](https://github.com/openSUSE/libsolv), the
SAT-based package dependency solver used by openSUSE, Fedora and others.

Upstream libsolv already understands Arch Linux repository databases, but its
default build uses RPM semantics. This fork builds with `-DARCHLINUX=ON`, which
switches libsolv to `DISTTYPE_ARCH` and Arch's version comparison rules
(`solv_vercmp_rpm_notilde`) instead of RPM's. The result is a libsolv that
behaves like pacman when comparing versions and resolving Arch packages.

The repository contains the upstream 0.7.41 source plus a small Arch layer:

| Path | Purpose |
| ---- | ------- |
| `arch/PKGBUILD` | Builds the `libsolv` package for Arch/Nexus |
| `arch/nexsolv-sync` | Converts pacman repository databases to `.solv` caches |
| `arch/nexsolv` | Human-readable dry-run solver CLI |

## Build and install

```sh
cd arch
makepkg -si
```

The package is named `libsolv` and has `pkgrel=2`, so it supersedes Arch extra's
`0.7.41-1` when the Nexus repository has priority. It installs the library,
headers, the usual libsolv tools (`archrepo2solv`, `archpkgs2solv`, `dumpsolv`,
...), Python bindings and the two `nexsolv` helpers.

## Usage

Build the solver caches from the pacman databases:

```sh
sudo nexsolv-sync
# or with a custom cache directory:
nexsolv-sync --output ~/.cache/nexsolv
```

`nexsolv` is a dry-run solver. It prints the transaction that pacman would
perform, but never downloads or changes packages:

```sh
nexsolv install bash
nexsolv remove bash
nexsolv upgrade
nexsolv dist-upgrade
nexsolv search editor
nexsolv info bash
nexsolv list
nexsolv repolist
```

If a transaction is unsolvable, `nexsolv` prints the libsolv problem report and
the available solutions. Pick one non-interactively with `--solution N`:

```sh
nexsolv --solution 1 install some-package
```

By default the CLI reads `/var/lib/pacman/sync` for repository caches,
`/var/cache/nexsolv` for `.solv` files and `/var/lib/pacman/local` for installed
packages. Override them with `--sync-dir`, `--cache-dir` and `--local-dir`, or
with `PACMAN_SYNC_DIR`, `NEXSOLV_CACHE_DIR` and `PACMAN_LOCAL_DIR`.

## Why not just Arch's libsolv?

Arch's package enables `ENABLE_ARCHREPO`, but builds with `-DFEDORA=1`, leaving
libsolv's default distribution type at RPM. That affects version comparison and
architecture policy. This fork makes Arch the default at compile time:

```cmake
-DARCHLINUX=ON
```

and adds a cache generator plus a small CLI that reads pacman's databases
directly, so the library is usable without writing a package manager first.

## Upstream

- Upstream: https://github.com/openSUSE/libsolv
- Version: 0.7.41
- License: BSD-3-Clause (see `LICENSE.BSD`)

The upstream source is kept at the repository root so patches can be carried
and rebased easily. Local changes belong in `arch/`.
