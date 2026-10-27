# thirp CLI

The installed command is `thirp`. A [GitHub Release](https://github.com/thirp/cloud/releases) attaches the `thirp-cli-0.1.0-*` archives that were built. The release notes name them.

| Archive | Machine |
|---|---|
| `thirp-cli-0.1.0-linux-amd64` | Linux x86_64 |
| `thirp-cli-0.1.0-darwin-amd64` | Intel Mac |
| `thirp-cli-0.1.0-darwin-arm64` | Apple Silicon Mac |
| `thirp-cli-0.1.0-windows-amd64.exe` | 64-bit Windows |

## Commands

```
usage: thirp  []
       thirp 
       thirp  
       thirp  
       thirp  
       thirp   SERVICE
       thirp   SERVICE [ SERVICE]  DURATION  TEXT [ ID]
       thirp   REQUEST_OR_GRANT
       thirp   GRANT
       thirp  SERVICE  127.0.0.1:PORT
       thirp   ENROLLMENT_TOKEN [ PATH]
global:  URL  (all commands; also ; else login-saved config; else )
         PATH
```

This list is generated from the implemented CLI. Documentation does not invent extra commands.

## Install

Verify `SHA256SUMS` before running a downloaded binary. These dev builds are unsigned.

Linux: the binary needs the distro OpenSSL 3 (`libssl.so.3`) and the system CA bundle. Put the file on `PATH` as `thirp`.

macOS (Intel or Apple Silicon): the binary links OpenSSL statically. A stock Mac does not need Homebrew or a separate OpenSSL install. Copy the file to a directory on `PATH` and `chmod 755` it. Gatekeeper may refuse an unsigned binary; for a dev build, remove the quarantine flag after you have checked the checksum (`xattr -d com.apple.quarantine thirp`) or allow it in System Settings. Certificate checks use the system trust store (Keychain anchors, then `/etc/ssl/cert.pem`).

Windows: the binary links OpenSSL statically and the CRT statically. Do not install OpenSSL. Certificate checks go through the Windows certificate chain engine, which can fetch a root that is not already in the store. Put `thirp.exe` on `PATH`.

## Session

The compiled default API origin is `https://api.thirp.io`. Resolution order is `--api-url`, then `THIRP_API_URL`, then the `api_url` saved by login, then that default. Pilot and a local control plane use the override, for example `--api-url https://pilot.thirp.net`. Local deploys pass `--api-url`. This repository does not activate DNS. `--organization` and `--environment` also read `THIRP_ORGANIZATION` and `THIRP_ENVIRONMENT`.

The session token is a file, on every OS. macOS Keychain and Windows Credential Manager are not used.

| OS | Directory | Session file |
|---|---|---|
| Linux and macOS | `$XDG_CONFIG_HOME/thirp`, or `~/.config/thirp` | mode `0600` |
| Windows | `%APPDATA%\thirp`, or `%USERPROFILE%\AppData\Roaming\thirp` | user-only ACL (current user, inheritance blocked) |

Non-secrets (`api_url`, organization, environment) go in `config` in that directory. `thirp logout` deletes the session file. Tokens are not printed.

`thirp login` binds `127.0.0.1:7645-7654` `/callback` when a browser identity provider is used. Where that flow is not enabled, use `thirp login --device`. It prints a user code and polls until approval. If the browser login-start is rejected, the CLI says to retry with `thirp login --device`.

## Connect versus enroll

`thirp connect` uses an authenticated session, finds an Active grant for that User and Service, issues a short-lived Caller credential, and listens locally. It does not create an access request.

`thirp agent enroll` writes `agent.token` (mode `0600`) and `agent.conf` for **`thirp-agent`** from a Thirp Runtime release.

Standalone `thirp-connect` is a Runtime binary with explicit Broker host, TLS, and token. It does not know about Organizations, Environments, or grants.

## Checksums

Release checksums and provenance for a published CLI build are listed under [releases/](../releases/README.md) and on the GitHub Release. Verify `SHA256SUMS` before running a downloaded binary.
