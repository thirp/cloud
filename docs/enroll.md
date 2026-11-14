# Agent enrollment

`thirp agent enroll` bootstraps files for the open-source Runtime Agent. It does not replace `thirp-agent`.

## Enroll

1. Create an Agent on the Environment (console or `POST /v1/organizations/{org_id}/environments/{env_id}/agents`) with a display name and Service IDs. Persist the enrollment token once.
2. Run `thirp agent enroll ENROLLMENT_TOKEN [--dir PATH] [--api-url URL] [--map SERVICE=127.0.0.1:PORT ...]`. `--api-url` is global. Resolution order is `--api-url`, then `THIRP_API_URL`, then the `api_url` saved by login, then `https://api.thirp.io`. A local control plane uses the override, for example `--api-url http://127.0.0.1:8080`. Local deploys pass `--api-url`.
3. Files written: `agent.token` mode `0600`, and `agent.conf` with `broker`, `token_file`, `tls_ca` pointing to `broker.crt`, `tls_server_name` when supplied, and the requested `map = SERVICE=IP:port` lines. The token is not inlined. The Broker CA PEM returned by exchange is written beside the config; an empty legacy exchange uses the hosted Broker CA roots. The enroll response supplies the Broker host; hosted names follow `<region>.broker.thirp.io`.
4. Run `thirp-agent` from a Thirp Runtime **0.16.6** (or later protocol-compatible) release with that config directory.

The source CLI accepts a repeated `--map SERVICE=IP:port` and writes each local TCP target. This source contract is pending release: existing pinned CLI 0.1.2 binaries are unchanged. The new console installer exchanges directly and installs the Runtime service without downloading a CLI. If the thing you want to publish is a log file, see [Publishing a log file as a Service](log_files.md).

Enrollment exchange is not idempotent. Persist the response. A used token cannot be exchanged again.

Enroll or run the one-step installer on each machine. Do not copy `agent.token`, `agent.conf`, or a CLI config directory onto another machine; those credentials are minted for the machine that runs enroll or the installer.

## Rotate

Issue a new Agent credential, then restart `thirp-agent` with the new token file. Overlapping credentials may both authenticate until the old one is revoked or expires.

## Revoke and uninstall

Revoke the Agent credential if it is compromised. The next AUTH fails.

To uninstall: stop `thirp-agent`, delete the token file and config directory, and revoke the Agent credential (disable the Agent if it should not re-enroll).

To replace the Runtime binary on an already-enrolled machine, run the verified installer with `--upgrade` (Unix) or `-Upgrade` (Windows), the matching platform, installer self-hash and the same Runtime URL, archive/agent SHA-256 pin flags as a fresh install (plus both OpenSSL DLL pins on Windows). Supply no enrollment token; credentials and maps stay unchanged. Upgrade waits up to 60 seconds for a new JSON `"event":"registered"` event indicating a Service is live on the broker session, and restores the previous Runtime on failure. The local `registered SERVICE_ID -> TARGET` map line is insufficient; reconnect events continue waiting within that window. Success retains one `.bak` per replaced file, replaced only after a later successful upgrade; `--uninstall` (Unix) or `-Uninstall` (Windows) removes those backups with the local install.
