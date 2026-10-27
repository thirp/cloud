# Agent enrollment

`thirp agent enroll` bootstraps files for the open-source Runtime Agent. It does not replace `thirp-agent`.

## Enroll

1. Create an Agent on the Environment (console or `POST /v1/organizations/{org_id}/environments/{env_id}/agents`) with a display name and Service IDs. Persist the enrollment token once.
2. Run `thirp agent enroll ENROLLMENT_TOKEN [--dir PATH] [--api-url URL]`. `--api-url` is global. Resolution order is `--api-url`, then `THIRP_API_URL`, then the `api_url` saved by login, then `https://api.thirp.io`. Pilot and a local control plane use the override, for example `--api-url https://pilot.thirp.net`. Local deploys pass `--api-url`.
3. Files written: `agent.token` mode `0600`, and `agent.conf` with `broker`, `token_file`, and optional TLS settings. The token is not inlined. There are no `map=` lines. The enroll response supplies the Broker host; hosted names follow `<region>.broker.thirp.io`.
4. Run `thirp-agent` from a Thirp Runtime **0.16.4** (or later protocol-compatible) release with that config directory.

The Agent still needs a local TCP target per Service (`--map` or `map =` in `agent.conf`; enroll does not write those lines). If the thing you want to publish is a log file, see [Publishing a log file as a Service](log_files.md).

Enrollment exchange is not idempotent. Persist the response. A used token cannot be exchanged again.

## Rotate

Issue a new Agent credential, then restart `thirp-agent` with the new token file. Overlapping credentials may both authenticate until the old one is revoked or expires.

## Revoke and uninstall

Revoke the Agent credential if it is compromised. The next AUTH fails.

To uninstall: stop `thirp-agent`, delete the token file and config directory, and revoke the Agent credential (disable the Agent if it should not re-enroll).
