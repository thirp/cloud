# Agents and MCP

Thirp's MCP tools let a vendor's AI client arrange customer-approved support access. The vendor's AI client calls MCP. The customer-side Thirp Agent is the enrolled `thirp-agent` running on the customer machine; `send_agent_install` emails the customer the installer and enrollment instructions for that setup.

MCP and the public OpenAPI are the same contract. Each MCP tool is one OpenAPI operation; `tools/list` builds each tool's input schema from that operation's parameters and JSON request body. See the [OpenAPI spec](https://api.thirp.io/v1/openapi.yaml), also published in this repository as [openapi/v1.yaml](../openapi/v1.yaml).

The ten tools support these steps:

- Recognize: `list_environments` lists environments the caller can see; `ensure_environment` selects or creates the named environment.
- Enroll: `send_agent_install` emails the Agent installer to the customer.
- Request: `request_access` creates an access request; `send_approve_link` emails the customer-side Approver the review link; `grant_status` reads grant status.
- Connect: `connect` returns connection details for an approved grant.
- Report: `get_report` fetches one support report; `list_audit` lists audit events.
- Revoke: `revoke_access` revokes an approved grant.

Thirp supplies connection primitives. Vendors author their own product checks and skills. The [example diagnose skill](examples/diagnose-skill.md) (`docs/examples/diagnose-skill.md`) is **EXAMPLE ONLY** — not the vendor's playbook, not a built-in Thirp diagnostic. There is no diagnose tool.

Connect an MCP client to `POST https://api.thirp.io/v1/mcp`. This is stateless MCP Streamable HTTP with JSON-RPC 2.0 JSON responses. It supports `initialize`, `notifications/initialized` (202, empty body), `ping`, `tools/list`, and `tools/call`; there are no server-initiated messages, GET event stream, or DELETE session endpoint. The supported protocol version is `2025-03-26`. Send `Content-Type: application/json` and `Accept: application/json, text/event-stream`; the server chooses JSON responses.

Use either the existing Organization Service Identity bearer credential for unattended hosts, or the person's existing signed-in Thirp user session. Both use `Authorization: Bearer …`; same-origin browser clients can use the existing HttpOnly session cookie. A browser Origin must match the configured public origin. Authenticate every MCP request. Authenticated GET and DELETE return 405 with `Allow: POST`. Do not put session tokens in chat, URLs, logs, or skill files. Integration roles/scopes, session roles/scopes, plan limits and grant ownership are the same as the corresponding `/v1` handlers. No new auth or grant model is introduced.

| Tool / OpenAPI operationId | Method and path | Purpose |
| --- | --- | --- |
| `list_environments` | GET `/v1/organizations/{org_id}/environments` | List environments the caller can see. |
| `ensure_environment` | POST `/v1/organizations/{org_id}/environment-selections` | Create the named environment if it is missing. |
| `send_agent_install` | POST `/v1/organizations/{org_id}/environments/{env_id}/agent-installs` | Email the Agent installer. |
| `request_access` | POST `/v1/organizations/{org_id}/environments/{env_id}/access-requests` | Create an access request. |
| `send_approve_link` | POST `/v1/organizations/{org_id}/access-requests/{id}/approve-link` | Email the Approver the review link. |
| `grant_status` | GET `/v1/organizations/{org_id}/access-requests/{id}` | Read grant status. |
| `connect` | POST `/v1/organizations/{org_id}/access-grants/{id}/credentials` | Return connection details for an approved grant. |
| `get_report` | GET `/v1/organizations/{org_id}/access-requests/{id}/report` | Fetch one support report. |
| `list_audit` | GET `/v1/organizations/{org_id}/audit-events` | List audit events. |
| `revoke_access` | POST `/v1/organizations/{org_id}/access-grants/{id}/revoke` | Revoke an approved grant. |

`tools/list` derives each `inputSchema` directly from that operation's OpenAPI parameters and JSON request-body schema. Parameters are top-level tool arguments; the HTTP JSON body is nested in the `body` argument. For example:

```json
{"jsonrpc":"2.0","id":1,"method":"initialize","params":{"protocolVersion":"2025-03-26","capabilities":{},"clientInfo":{"name":"vendor-agent","version":"1"}}}
{"jsonrpc":"2.0","id":2,"method":"tools/call","params":{"name":"request_access","arguments":{"org_id":"ORGANIZATION_UUID","env_id":"ENVIRONMENT_UUID","body":{"catalog_service_ids":["CATALOG_SERVICE_UUID"],"duration_seconds":600,"reason":"Diagnose case 42","external_case_id":"CASE-42"}}}}
```

Tool success contains the same JSON as HTTP in `content[0].text`. Existing API errors use `isError: true` with the existing error body. Unknown tools and malformed RPC/arguments use JSON-RPC errors. There is no generic call/run tool and no approval tool.

`ensure_environment` requires `name` and `region_id`. When `external_deployment_id` is supplied, it selects by the pair (`external_customer_id`, `external_deployment_id`); otherwise it selects by trimmed exact name. It pages through visible Environments. Selected results are 200, created results 201. This separate selection noun preserves the existing non-idempotent create operation. Selection and creation run under the existing API write mutex; use the existing single control API writer/database deployment model.

`list_environments` keeps the existing list envelope and fields. An enrolled Agent with a live Broker registration makes its Environment online; an enrolled Agent without one makes it offline; no enrolled Agent means missing-agent. Services are catalog objects including wire `service_id` and UUID `id`. Install uses wire Service IDs; access requests use catalog UUIDs. `grant_status` adds `status` and `grants` while preserving the existing request `state`. A future approved window is pending until its start; any active member makes a multi-Service request active. Inspect individual grants for mixed revoked/expired members.

Install mail uses the configured **Thirp From**. Reply-To is the requesting user's `verified_email`; an unattended Service Identity uses an active Organization Owner's verified email because Organizations have no contact email field. Both mail kinds name the vendor Organization in the body. Approve mail uses the durable notification outbox and persists Reply-To across retries. Install mail is synchronous through the same configured transport: the outbox persists plaintext bodies, so queuing an enrollment code there would violate hash-only enrollment storage. Delivery failure revokes the minted enrollment; retry by calling `send_agent_install` again. The code is absent from API/MCP responses and access logs, and the enrollment table stores its hash. On Linux, Resend passes the mail body to its existing script through an inherited anonymous memory descriptor; no plaintext body file is created. File capture is local development only and writes private files.

The vendor supplies the Environment's `os` and `arch` from knowledge of the customer host. Never infer them from the browser, mail client, or the laptop opening the email. `ensure_environment`, Environment create/PATCH, and `send_agent_install` accept the optional pair and save new values, including when selection finds an existing Environment. Omit both keys to reuse the stored values. Partial, null, empty, unknown pairs and extra properties on `ensure_environment` or `send_agent_install` refuse. Creating or selecting an Environment without a platform is allowed; sending install mail without a supplied or stored platform refuses and sends no mail. GET and `list_environments` return `os` and `arch`, each `null` until set.

Only the following four pairs are allowed, including `darwin` + `arm64`. Install mail includes the matching `thirp` CLI and `thirp-agent` Runtime archive URLs and SHA-256 values, followed by the existing `thirp agent enroll … --api-url …` command.

| os | arch | CLI URL and SHA-256 | Runtime archive URL and SHA-256 | thirp-agent SHA-256 |
| --- | --- | --- | --- | --- |
| `linux` | `amd64` | [CLI](https://github.com/thirp/cloud/releases/download/cli-0.1.3/thirp-cli-0.1.3-linux-amd64)<br>`9dba7acbfb747992b07bd5ddc0cd056a1dc90dd974ed93ee5b410546740a30e8` | [Runtime](https://github.com/thirp/runtime/releases/download/v0.16.8/thirp-runtime-linux-x86_64-0.16.8.tar.gz)<br>`8ea0683cafe468032d2ddc050dd44a0ac177154e2db59cd1c89a5f22b4b370b1` | `7a8e7c34352a52cbef7f49e5349b47ec8759d16f89edcd7d709bb15bf623ee6f` |
| `darwin` | `amd64` | [CLI](https://github.com/thirp/cloud/releases/download/cli-0.1.3/thirp-cli-0.1.3-darwin-amd64)<br>`06126f2e94b78a261387f018e34e1bd355cbdb9ff9a60b3f39677bebaf9e6930` | [Runtime](https://github.com/thirp/runtime/releases/download/v0.16.8/thirp-runtime-macos-x86_64-0.16.8.tar.gz)<br>`4b34ed120c0ce8e24f5fd26cd91f60adb6c31d1295bf1ae99a93a399504d8e79` | `5dd3961b1ba312668167b2fc6a3501a5d248d22cd60109876728378ff7b91bec` |
| `darwin` | `arm64` | [CLI](https://github.com/thirp/cloud/releases/download/cli-0.1.3/thirp-cli-0.1.3-darwin-arm64)<br>`567ec4ab5d7073f3784a596f53fc73a0ec31e667896e29e0d0626655c2ff95ac` | [Runtime](https://github.com/thirp/runtime/releases/download/v0.16.8/thirp-runtime-macos-arm64-0.16.8.tar.gz)<br>`1cd73070c4aadf505e0f1912201299058f78bec0c25ed281dc1e5143c786666e` | `7b16bc728af661621bbcdc11b99456feafbba67076532cc227ec3b5fa68e1666` |
| `windows` | `amd64` | [CLI](https://github.com/thirp/cloud/releases/download/cli-0.1.3/thirp-cli-0.1.3-windows-amd64.exe)<br>`30ecf15cfd222d3cf4b245d69f804e8e3900b615f9d7ae5f5456b8d7a59140bb` | [Runtime](https://github.com/thirp/runtime/releases/download/v0.16.8/thirp-runtime-windows-AMD64-0.16.8.zip)<br>`ec43f8ebca896cd9f532138783fac63d74f9cee7f182ddb395bf1db58fcd1fbb` | `77b365330de3f5ac3e583492091ceaac1e40b39d310f988067aac4b564f2d72a` |

CLI checksums come from the public release's [SHA256SUMS](https://github.com/thirp/cloud/releases/download/cli-0.1.3/SHA256SUMS). Runtime pins verify each downloaded archive; the inner `SHA256SUMS` verifies its extracted tree, including `thirp-agent`. On Windows the Runtime zip also contains OpenSSL 3 DLLs whose SHA-256 values are `0242e31713fa090d75c0cc2e17c4ee2cbc3a27ca53f7a45729f30983d593bcb1` (`libssl-3-x64.dll`) and `81bc67d537fcd320d8ede1c5a49506629f0fd04e42ec7e9b0bab70a822d4fba5` (`libcrypto-3-x64.dll`). No download is performed by the API or replay. Linux CLI needs distro OpenSSL 3; macOS and Windows CLI link OpenSSL statically. As [enrollment](enroll.md) requires, the customer runs a compatible `thirp-agent` and configures local TCP targets; enrollment authorizes Service names and does not invent target addresses.

Repeat `send_agent_install` with the **same display_name** (default `customer-agent`) and additional wire Service IDs to expand that Agent's authorization when the new one-use code is exchanged. The supplied-or-stored platform rule still applies on repeat calls. Existing Services remain authorized. Added IDs are staged on the existing enrollment and do not become authorized before exchange. Reenroll and restart/configure the Runtime Agent for the additional local targets. There is no ensure_services primitive.

An Environment needs an active Approver membership before `send_approve_link`; if none has a verified email the operation refuses. Use the existing invitation API to establish customer Approvers; install mail does not assign approval authority.

Approval mail's primary and alternate links both point to the existing console request deep link. A signed-out GET redirects to sign-in; a signed-in GET only renders the request. Approval requires the Approver's explicit POST through the existing handler, with its scope/authority checks, SameSite session cookie and same-origin browser POST check. Integration identities are refused even if vendor approval was delegated. An MCP call or chat yes cannot approve.

MCP cannot carry the TCP stream. `connect` returns private bounded material for the existing CLI/SDK Broker path. It refuses before the window, after expiry, and after revoke. The Broker resets streams at grant expiry/revoke using its existing policy/lease behavior. Do not print returned Caller tokens. Report data uses the same encoders as the console, including request/grant audit events so a denied decision still names the decider. Report audit facts require the existing audit-reader role (Administrator is the usual unattended role). Audit retention is the existing 90 days.
