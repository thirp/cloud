# Agents and MCP

Thirp's MCP tools let an AI client arrange customer-approved support access or manage a Cloud for Platforms account. The vendor's AI client calls MCP. The customer-side Thirp Agent is the enrolled `thirp-agent` running on the customer machine; `send_agent_install` emails the customer the installer and enrollment instructions for that setup.

MCP and the public OpenAPI are the same contract. Each MCP tool is one OpenAPI operation; `tools/list` builds each tool's input schema from that operation's parameters and JSON request body. See the [OpenAPI spec](https://api.thirp.io/v1/openapi.yaml), also published in this repository as [openapi/v1.yaml](../openapi/v1.yaml).

## Cloud for Support tools

Eleven tools support the Cloud for Support workflow and Agent setup.

- Recognize: `list_environments` lists environments the caller can see; `ensure_environment` selects or creates the named environment.
- Enroll: `send_agent_install` emails the Agent installer to the customer. `send_platform_agent_install` emails one-step install instructions for an account-scoped Agent.
- Request: `request_access` creates an access request; `send_approve_link` emails the customer-side Approver the review link; `grant_status` reads grant status.
- Connect: `connect` returns connection details for an approved grant.
- Report: `get_report` fetches one support report; `list_audit` lists audit events.
- Revoke: `revoke_access` revokes an approved grant.

Thirp supplies connection primitives. Vendors author their own product checks and skills. The [example diagnose skill](examples/diagnose-skill.md) (`docs/examples/diagnose-skill.md`) is **EXAMPLE ONLY** — not the vendor's playbook, not a built-in Thirp diagnostic. There is no diagnose tool.

Connect an MCP client to `POST https://api.thirp.io/v1/mcp`. This is stateless MCP Streamable HTTP with JSON-RPC 2.0 JSON responses. It supports `initialize`, `notifications/initialized` (202, empty body), `ping`, `tools/list`, and `tools/call`; there are no server-initiated messages, GET event stream, or DELETE session endpoint. The supported protocol version is `2025-03-26`. Send `Content-Type: application/json` and `Accept: application/json, text/event-stream`; the server chooses JSON responses.

Use either the existing Organization Service Identity bearer credential for unattended hosts, or the person's existing signed-in Thirp user session. Both use `Authorization: Bearer …`; same-origin browser clients can use the existing HttpOnly session cookie. A browser Origin must match the configured public origin. Authenticate every MCP request. Authenticated GET and DELETE return 405 with `Allow: POST`. Do not put session tokens in chat, URLs, logs, or skill files. Integration roles/scopes, session roles/scopes, plan limits and grant ownership are the same as the corresponding `/v1` handlers. No new auth or grant model is introduced.

| Tool / OpenAPI operationId | Method and path | Purpose | readOnlyHint | destructiveHint | idempotentHint |
| --- | --- | --- | --- | --- | --- |
| `list_environments` | GET `/v1/organizations/{org_id}/environments` | List environments the caller can see. | true | false | true |
| `ensure_environment` | POST `/v1/organizations/{org_id}/environment-selections` | Create the named environment if it is missing. | false | false | true |
| `send_platform_agent_install` | POST `/v1/organizations/{org_id}/agent-installs` | Email one-step instructions for an account-scoped Agent. | false | false | false |
| `send_agent_install` | POST `/v1/organizations/{org_id}/environments/{env_id}/agent-installs` | Email the Agent installer. | false | false | false |
| `request_access` | POST `/v1/organizations/{org_id}/environments/{env_id}/access-requests` | Create an access request. | false | false | false |
| `send_approve_link` | POST `/v1/organizations/{org_id}/access-requests/{id}/approve-link` | Email the Approver the review link. | false | false | false |
| `grant_status` | GET `/v1/organizations/{org_id}/access-requests/{id}` | Read grant status. | true | false | true |
| `connect` | POST `/v1/organizations/{org_id}/access-grants/{id}/credentials` | Return connection details for an approved grant. | false | true | false |
| `get_report` | GET `/v1/organizations/{org_id}/access-requests/{id}/report` | Fetch one support report. | true | false | true |
| `list_audit` | GET `/v1/organizations/{org_id}/audit-events` | List audit events. | true | false | true |
| `revoke_access` | POST `/v1/organizations/{org_id}/access-grants/{id}/revoke` | Revoke an approved grant. | false | true | true |

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
| `linux` | `amd64` | [CLI](https://github.com/thirp/cloud/releases/download/cli-0.1.5/thirp-cli-0.1.5-linux-amd64)<br>`c323de1c35a6e202ef216492d04d3b0babffdb6bc2fc49f5e65c87dfe12d189f` | [Runtime](https://github.com/thirp/runtime/releases/download/v0.16.10/thirp-runtime-linux-x86_64-0.16.10.tar.gz)<br>`9488ec6f558ed6ca68ec321f1bd5948b9efa37421f419f4681105f5935c42f4d` | `82867dd4f3b16ec6355d0eb9eeec45563b262dfd73708c8e3bdb1d0fe3c8b419` |
| `darwin` | `amd64` | [CLI](https://github.com/thirp/cloud/releases/download/cli-0.1.5/thirp-cli-0.1.5-darwin-amd64)<br>`24d92fc420932307319bf6a19067df51fe93e7748f3c23f4b5b90b1ebb1143b5` | [Runtime](https://github.com/thirp/runtime/releases/download/v0.16.10/thirp-runtime-macos-x86_64-0.16.10.tar.gz)<br>`5e70023c0a61633598f092271fb87d9a89523bcf0a01d3a2cccb078cd4d22f20` | `93928142cb14b88dfddc5cafac6b063a3e32437dbe4a1803835f7b1ff3fa27db` |
| `darwin` | `arm64` | [CLI](https://github.com/thirp/cloud/releases/download/cli-0.1.5/thirp-cli-0.1.5-darwin-arm64)<br>`e08c2b190fa944e6cf20938d921d8d13ed8ad4fa855153205aa582ec461149a8` | [Runtime](https://github.com/thirp/runtime/releases/download/v0.16.10/thirp-runtime-macos-arm64-0.16.10.tar.gz)<br>`788974b49db02b0952faba644b2360ab22200a1709763a9a5268b25c330456c6` | `76a609c3101005d01922492e287e35e42690e3355bb9210d41a249f2a99560bb` |
| `windows` | `amd64` | [CLI](https://github.com/thirp/cloud/releases/download/cli-0.1.5/thirp-cli-0.1.5-windows-amd64.exe)<br>`04314f5d2e5b80f0f764015b4687d7cb051718f75c21cc00c7d1eac644642b7c` | [Runtime](https://github.com/thirp/runtime/releases/download/v0.16.10/thirp-runtime-windows-AMD64-0.16.10.zip)<br>`41a86af32052ef2819808dfd25d4a4bfaf49209a0dfa270a0b537c26537cb8b3` | `cca4b62c1f1867d2926b3dea6b094f32d783cdbb68df25ebb51c6ad5f3e9161f` |

CLI checksums come from the public release's [SHA256SUMS](https://github.com/thirp/cloud/releases/download/cli-0.1.5/SHA256SUMS). Runtime pins verify each downloaded archive; the inner `SHA256SUMS` verifies its extracted tree, including `thirp-agent`. On Windows the Runtime zip also contains OpenSSL 3 DLLs whose SHA-256 values are `0242e31713fa090d75c0cc2e17c4ee2cbc3a27ca53f7a45729f30983d593bcb1` (`libssl-3-x64.dll`) and `81bc67d537fcd320d8ede1c5a49506629f0fd04e42ec7e9b0bab70a822d4fba5` (`libcrypto-3-x64.dll`). No download is performed by the API or replay. Linux CLI needs distro OpenSSL 3; macOS and Windows CLI link OpenSSL statically. As [enrollment](enroll.md) requires, the customer runs a compatible `thirp-agent` and configures local TCP targets; enrollment authorizes Service names and does not invent target addresses.

Repeat `send_agent_install` with the **same display_name** (default `customer-agent`) and additional wire Service IDs to expand that Agent's authorization when the new one-use code is exchanged. The supplied-or-stored platform rule still applies on repeat calls. Existing Services remain authorized. Added IDs are staged on the existing enrollment and do not become authorized before exchange. Reenroll and restart/configure the Runtime Agent for the additional local targets. There is no ensure_services primitive.

An Environment needs an active Approver membership before `send_approve_link`; if none has a verified email the operation refuses. Use the existing invitation API to establish customer Approvers; install mail does not assign approval authority.

Approval mail's primary and alternate links both point to the existing console request deep link. A signed-out GET redirects to sign-in; a signed-in GET only renders the request. Approval requires the Approver's explicit POST through the existing handler, with its scope/authority checks, SameSite session cookie and same-origin browser POST check. Integration identities are refused even if vendor approval was delegated. An MCP call or chat yes cannot approve.

MCP cannot carry the TCP stream. `connect` returns private bounded material for the existing CLI/SDK Broker path. It refuses before the window, after expiry, and after revoke. The Broker resets streams at grant expiry/revoke using its existing policy/lease behavior. Do not print returned Caller tokens. Report data uses the same encoders as the console, including request/grant audit events so a denied decision still names the decider. Report audit facts require the existing audit-reader role (Administrator is the usual unattended role). Audit retention is the existing 90 days.


## Cloud for Platforms tools

A person signs up and pays on thirp.net before using the tools to configure a Platforms account. Use `send_platform_agent_install` to email the install instructions. A person runs them on the Agent host.

| Tool / OpenAPI operationId | Method and path | Purpose | readOnlyHint | destructiveHint | idempotentHint |
| --- | --- | --- | --- | --- | --- |
| `list_services` | GET `/v1/organizations/{org_id}/services` | List configured Services, including archived entries. | true | false | true |
| `create_service` | POST `/v1/organizations/{org_id}/services` | Create a Service with `service_id` and `display_name`. Omit `environment_id` for Platforms. | false | false | false |
| `get_service` | GET `/v1/organizations/{org_id}/services/{id}` | Read a Service's catalog state and live registration. | true | false | true |
| `list_agents` | GET `/v1/organizations/{org_id}/agents` | List Agents in the account. | true | false | true |
| `get_agent` | GET `/v1/organizations/{org_id}/agents/{id}` | Check whether an Agent is online and when it was last seen. | true | false | true |
| `get_usage` | GET `/v1/organizations/{org_id}/usage` | Read usage and quotas, with optional `from` and `to` timestamps. | true | false | true |
| `get_organization` | GET `/v1/organizations/{org_id}` | Read the account plan and default region ID. | true | false | true |
| `get_region` | GET `/v1/regions/{id}` | Read a region's public endpoint and operational status. | true | false | true |
| `list_caller_enrollments` | GET `/v1/organizations/{org_id}/caller-enrollments` | List visible Caller enrollments without tokens. | true | false | true |
| `revoke_caller_enrollment` | POST `/v1/organizations/{org_id}/caller-enrollments/{enrollment_id}/revoke` | Revoke an enrollment and grants it alone holds. | false | true | true |

Each row shows the annotations that `tools/list` reads from OpenAPI. The two Caller enrollment tools require a user login session. An Owner, Administrator, or the enrolling user can revoke an enrollment.

For Platforms connect info, get `default_region_id` from `get_organization`, then get `public_endpoint` and `operational_status` from `get_region`. The tools return connection metadata without credentials. A person at the keyboard runs `thirp caller enroll` once on the machine that will call the Service. The enrolled machine opens the link with `thirp-connect --caller-conf PATH`. For a short-lived connection, the person uses the console or `thirp connect`. The CLI opens the stream.
