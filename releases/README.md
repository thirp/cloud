# CLI releases

GitHub Releases attach the `thirp` CLI archives that were built for that release. They do not attach hosted server binaries. The release notes name the archives. `darwin-arm64` is not attached until that Mac build exists.

| Archive | When it is built |
|---|---|
| `thirp-cli-0.1.0-linux-amd64` | Linux build, system OpenSSL 3 |
| `thirp-cli-0.1.0-darwin-amd64` | Intel Mac build, static OpenSSL |
| `thirp-cli-0.1.0-windows-amd64.exe` | Windows build, static OpenSSL |

Checksums for a built archive are written next to this file when the publisher runs with `--build-cli`. Verify `SHA256SUMS` on the GitHub Release before running the binary.

The CLI archive includes TERMS and NOTICE. It must not include the Thirp Runtime Apache-2.0 LICENSE as if the Cloud CLI were open source.
