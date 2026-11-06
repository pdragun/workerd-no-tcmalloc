# workerd without tcmalloc (linux-arm64)

Unofficial builds of upstream [cloudflare/workerd](https://github.com/cloudflare/workerd), unpatched,
compiled with `--@workerd//src/workerd/server:use_tcmalloc=False`.

## Why

The official linux-arm64 workerd (the npm `@cloudflare/workerd-linux-arm64` package that Wrangler and
Miniflare use) links tcmalloc, which needs a 48-bit virtual address space. On arm64 kernels built with a
39-bit address space it stops at startup with

```text
MmapAligned() failed ... TCMalloc assumes a 48-bit virtual address space
```

([cloudflare/workerd#5020](https://github.com/cloudflare/workerd/issues/5020)), so `wrangler dev` and the
Workers Vitest pool can't run. Without tcmalloc, workerd uses glibc's malloc and starts normally.

This is for local development only: test and deploy production builds on the official workerd. The repo
stops being needed once #5020 is fixed upstream.

## Does your machine need it?

```sh
zcat /proc/config.gz | grep ARM64_VA_BITS    # CONFIG_ARM64_VA_BITS_39=y -> yes
```

or you get the `MmapAligned() failed` error above.

## Tested on

- ChromeOS 156, Linux development environment (Crostini), Debian 13 (trixie), glibc 2.41, kernel 6.6,
  Qualcomm Snapdragon 7c-class CPU (Kryo 4XX)
- A Docker container on that machine: Debian 12 (bookworm), glibc 2.36

A Worker project's `wrangler dev`, D1 migrations, `wrangler types`, the Workers Vitest pool and
Playwright tests against the dev server all worked.

## Should also work (untested)

Any arm64 Linux with glibc and a kernel with a 39-bit address space, for example:

- other ARM Chromebooks (MediaTek or Qualcomm) in the Linux development environment
- Raspberry Pi OS 64-bit, on kernels built with a 39-bit address space
- Android devices, inside a glibc distribution under Termux's proot-distro (not plain Termux, which uses
  Android's bionic libc)

Reports, working or not, are welcome in Issues.

## Builds

Each [release](../../releases) is tagged with the workerd version it was built from (such as
`v1.20261102.1`) and holds the binary and its sha256. **Use the newest `workerd` in your
`package-lock.json`**: a project can lock several (Wrangler, the Vitest pool and the Vite plugin each
pull in their own), and `MINIFLARE_WORKERD_PATH` replaces them all, so match the newest. Another
version may not work.

```sh
npm ls workerd --all    # pick the highest version listed
```

```sh
gh release download v1.20261102.1 -R pdragun/workerd-no-tcmalloc
sha256sum -c workerd.sha256
chmod +x workerd
export MINIFLARE_WORKERD_PATH=$PWD/workerd
```

To build another version, run the "Build workerd arm64 (no tcmalloc)" workflow with that tag and tick
"release". A cold build takes about 1–4 hours; when a run times out, "Re-run jobs" resumes it from the
cache. The release is made only when the binary passes a smoke test under an emulated 39-bit address
space.

The workflow is adapted from
[coolcomfort/workerd-android-arm64](https://github.com/coolcomfort/workerd-android-arm64), with every
action and bazelisk pinned.
