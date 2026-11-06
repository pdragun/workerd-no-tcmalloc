# workerd without tcmalloc (linux-arm64)

Builds upstream [cloudflare/workerd](https://github.com/cloudflare/workerd) at a release tag, unpatched, with
`--@workerd//src/workerd/server:use_tcmalloc=False`, so it runs on arm64 kernels with a 39-bit virtual address space
(cloudflare/workerd#5020). The workflow is adapted from
[coolcomfort/workerd-android-arm64](https://github.com/coolcomfort/workerd-android-arm64), with its actions and
bazelisk pinned.

Run it from Actions with the tag of the workerd version your Wrangler uses, download the artifact, and point
`MINIFLARE_WORKERD_PATH` at the binary.
