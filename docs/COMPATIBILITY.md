# Compatibility

Thirp Cloud CLI versions and Thirp Runtime versions are independent. They are not lockstepped because the control plane vendors Runtime artifacts.

| CLI | Public API | Thirp Runtime | Protocol | Minimum Agent |
|---|---|---|---|---|
| 0.1.0 | 1.0 | 0.16.0 | 1.0 | 0.16.0 |

The Agent is the Runtime `thirp-agent` binary from a Runtime 0.16.0 (or later protocol-compatible) release. Protocol 1.0 wire behavior is unchanged from that Runtime release.

Windows and macOS `thirp` CLI archives (`thirp-cli-<version>-windows-amd64.exe`, `thirp-cli-<version>-darwin-arm64`) are reserved names. They are not shipped in 0.1.0.
