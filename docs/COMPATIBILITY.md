# Compatibility

Thirp Cloud CLI versions and Thirp Runtime versions are independent. They are not lockstepped because the control plane vendors Runtime artifacts.

| CLI | Public API | Thirp Runtime | Protocol | Minimum Agent |
|---|---|---|---|---|
| 0.1.3 | 1.0 | 0.16.6 | 1.0 | 0.16.6 |

The Agent is the Runtime `thirp-agent` binary from a Runtime 0.16.6 (or later protocol-compatible) release. Protocol 1.0 wire behavior is unchanged from that Runtime release.

A 0.1.3 release attaches the `thirp-cli-0.1.3-*` archives that were built. The release notes name them. Linux links the system OpenSSL 3. macOS and Windows link OpenSSL statically, so those binaries do not need Homebrew or an OpenSSL install. `darwin-arm64` is not attached until that Mac build exists.
