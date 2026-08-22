# thirp CLI

The installed command is `thirp`. First public archive: `thirp-cli-0.1.0-linux-amd64` from [GitHub Releases](https://github.com/thirp/cloud/releases).

Windows and macOS builds are not part of this release.

## Commands

```
usage: thirp login [--device] [--api-url URL]
       thirp logout
       thirp organizations list
       thirp environments list
       thirp services list
       thirp services status SERVICE
       thirp access request SERVICE --duration DURATION --reason TEXT [--case ID]
       thirp access status REQUEST_OR_GRANT
       thirp access revoke GRANT
       thirp connect SERVICE --listen 127.0.0.1:PORT
       thirp agent enroll ENROLLMENT_TOKEN [--dir PATH]
```

This list is generated from the implemented CLI. Documentation does not invent extra commands.

## Session

Production API origin: `https://api.thirp.net`. `--api-url`, `--organization`, and `--environment` also read `THIRP_API_URL`, `THIRP_ORGANIZATION`, and `THIRP_ENVIRONMENT`. The compiled default remains `http://127.0.0.1:8080` for local use.

The session token is stored with `secret-tool` when that binary exists, otherwise `XDG_CONFIG_HOME/thirp/session` (or `~/.config/thirp/session`) mode `0600`. Non-secrets (`api_url`, organization, environment) go in `config` under the same directory. Tokens are not printed.

`thirp login` binds `127.0.0.1:7645-7654` `/callback` when a browser identity provider is used. `thirp login --device` prints a user code and polls until approval.

## Connect versus enroll

`thirp connect` uses an authenticated session, finds an Active grant for that User and Service, issues a short-lived Caller credential, and listens locally. It does not create an access request.

`thirp agent enroll` writes `agent.token` (mode `0600`) and `agent.conf` for **`thirp-agent`** from a Thirp Runtime release.

Standalone `thirp-connect` is a Runtime binary with explicit Broker host, TLS, and token. It does not know about Organizations, Environments, or grants.

## Checksums

Release checksums and provenance for a published CLI build are listed under [releases/](../releases/README.md) and on the GitHub Release. Verify `SHA256SUMS` before running a downloaded binary.
