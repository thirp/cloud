# CLI releases

GitHub Releases attach the `thirp` CLI. They do not attach hosted server binaries.

| Archive | Status |
|---|---|
| `thirp-cli-0.1.0-linux-amd64` | First public CLI |
| `thirp-cli-<version>-windows-amd64.exe` | Reserved; not shipped |
| `thirp-cli-<version>-darwin-arm64` | Reserved; not shipped |

Checksums for a built archive are written next to this file when the publisher runs with `--build-cli`. Verify `SHA256SUMS` on the GitHub Release before running the binary.

The CLI archive includes TERMS and NOTICE. It must not include the Thirp Runtime Apache-2.0 LICENSE as if the Cloud CLI were open source.
