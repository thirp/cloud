# Getting started

This how-to takes a vendor from a Cloud for Support trial to a customer-approved connection to a named Service. Choose [Console](#console) to arrange access in the browser, or [Agentic](#agentic) to have the vendor's AI client arrange it through MCP. Both paths use the same Organization, customer Environment, Approver, Service definitions, and enrolled Runtime process.

The vendor's AI client calls MCP. The customer-side Thirp Agent is the outbound `thirp-agent` process running on the customer host. Installing or configuring the AI client does not install that customer-side process.

The [console](https://api.thirp.io/console/) presents the public API. The [live OpenAPI contract](https://api.thirp.io/v1/openapi.yaml) describes the API operations; [Agents and MCP](agents.md) describes the MCP contract. Keep enrollment and connection credentials private.

## Console

### 1. Start a trial and sign in

Open [Start a trial](https://api.thirp.io/console/signup) and sign in. Complete the organization form with your company name and organization slug, and accept the terms if they apply to your business. Work in that vendor Organization for the remaining steps. Return to the [console](https://api.thirp.io/console/) for subsequent sessions.

### 2. Create the customer Environment

In **Environments**, use **Create Environment**. Enter the **Customer/site name** and **Region**. The optional **Vendor customer reference** and **Deployment reference** associate the Environment with identifiers you already use for this customer and deployment. Open the resulting Environment and confirm that it represents the customer host you intend to support.

### 3. Invite a customer Approver

In **Members**, use the invitation form. Supply the customer's **Email**, choose **Approver** as the **Role**, and select this **Environment**. Send the invitation and have the customer sign in and accept it. Use customer approval for each request in this walkthrough; the customer controls the Environment's approval policy.

An Approver's membership is scoped to the assigned Environment. Invitation links do not contain tokens in the URL. See [Customer Approver](approver.md) for the authority and policy details. Enrolling an Agent does not make its installer an Approver.

### 4. Create the named Service

Use the Environment's **Create Service** form. Enter a **Service name shown to people** and a **Stable Service identifier**, with an optional **Description**. Choose a stable identifier for the local TCP service you intend to expose. Repeat for each named Service needed for the support session.

A Service definition names what can be requested; the local port is configured on the customer host in step 8. For a log file, follow [Publishing a log file as a Service](log_files.md) to provide a TCP target.

### 5. Set the customer host's OS and architecture

Determine the platform from knowledge of the customer host. In the Environment, set **Operating system** and **Architecture**, then use **Save platform**. These fields are also available when creating an Environment. Use one of the four pairs in the table below; `darwin` means macOS and `amd64` means x86_64.

Do not infer the platform from the browser, mail client, or machine opening an email. If you do not know the customer host's platform, obtain that information before choosing downloads or sending installer mail.

### 6. Install the matching CLI and Runtime on the customer host

Download both assets from the row matching the Environment's OS and architecture. Verify the CLI binary and Runtime archive against their SHA-256 values before running them. Install the CLI on the customer host as `thirp` (`thirp.exe` on Windows) on `PATH`. Extract the Runtime archive to obtain `thirp-agent`; that is the process you will run on this host.

| os | arch | CLI URL and SHA-256 | Runtime archive URL and SHA-256 | thirp-agent SHA-256 |
| --- | --- | --- | --- | --- |
| `linux` | `amd64` | [CLI](https://github.com/thirp/cloud/releases/download/cli-0.1.3/thirp-cli-0.1.3-linux-amd64)<br>`9dba7acbfb747992b07bd5ddc0cd056a1dc90dd974ed93ee5b410546740a30e8` | [Runtime](https://github.com/thirp/runtime/releases/download/v0.16.8/thirp-runtime-linux-x86_64-0.16.8.tar.gz)<br>`8ea0683cafe468032d2ddc050dd44a0ac177154e2db59cd1c89a5f22b4b370b1` | `7a8e7c34352a52cbef7f49e5349b47ec8759d16f89edcd7d709bb15bf623ee6f` |
| `darwin` | `amd64` | [CLI](https://github.com/thirp/cloud/releases/download/cli-0.1.3/thirp-cli-0.1.3-darwin-amd64)<br>`06126f2e94b78a261387f018e34e1bd355cbdb9ff9a60b3f39677bebaf9e6930` | [Runtime](https://github.com/thirp/runtime/releases/download/v0.16.8/thirp-runtime-macos-x86_64-0.16.8.tar.gz)<br>`4b34ed120c0ce8e24f5fd26cd91f60adb6c31d1295bf1ae99a93a399504d8e79` | `5dd3961b1ba312668167b2fc6a3501a5d248d22cd60109876728378ff7b91bec` |
| `darwin` | `arm64` | [CLI](https://github.com/thirp/cloud/releases/download/cli-0.1.3/thirp-cli-0.1.3-darwin-arm64)<br>`567ec4ab5d7073f3784a596f53fc73a0ec31e667896e29e0d0626655c2ff95ac` | [Runtime](https://github.com/thirp/runtime/releases/download/v0.16.8/thirp-runtime-macos-arm64-0.16.8.tar.gz)<br>`1cd73070c4aadf505e0f1912201299058f78bec0c25ed281dc1e5143c786666e` | `7b16bc728af661621bbcdc11b99456feafbba67076532cc227ec3b5fa68e1666` |
| `windows` | `amd64` | [CLI](https://github.com/thirp/cloud/releases/download/cli-0.1.3/thirp-cli-0.1.3-windows-amd64.exe)<br>`30ecf15cfd222d3cf4b245d69f804e8e3900b615f9d7ae5f5456b8d7a59140bb` | [Runtime](https://github.com/thirp/runtime/releases/download/v0.16.8/thirp-runtime-windows-AMD64-0.16.8.zip)<br>`ec43f8ebca896cd9f532138783fac63d74f9cee7f182ddb395bf1db58fcd1fbb` | `77b365330de3f5ac3e583492091ceaac1e40b39d310f988067aac4b564f2d72a` |

CLI checksums come from the public cli-0.1.3 [SHA256SUMS](https://github.com/thirp/cloud/releases/download/cli-0.1.3/SHA256SUMS). Runtime pins verify the downloaded archive; its inner `SHA256SUMS` verifies the extracted tree, including `thirp-agent`. On Windows the Runtime zip also contains OpenSSL 3 DLLs whose SHA-256 values are `0242e31713fa090d75c0cc2e17c4ee2cbc3a27ca53f7a45729f30983d593bcb1` (`libssl-3-x64.dll`) and `81bc67d537fcd320d8ede1c5a49506629f0fd04e42ec7e9b0bab70a822d4fba5` (`libcrypto-3-x64.dll`). See [CLI installation](cli.md#install) for executable permissions, platform dependencies, and checksum guidance.

### 7. Enroll the customer-side Thirp Agent

In the Environment's Agent enrollment form, supply an Agent name and select the Services this Agent will serve. Preserve the one-use enrollment token privately and run the enrollment command on the customer host:

```text
thirp agent enroll ENROLLMENT_TOKEN --api-url https://api.thirp.io
```

Replace `ENROLLMENT_TOKEN` with the issued token. `thirp agent enroll` writes `agent.token` and `agent.conf` for the Runtime process. It does not replace `thirp-agent`, start it, or configure its local Service targets. The token exchange cannot be repeated with a used token. See [Agent enrollment](enroll.md) and [CLI: connect versus enroll](cli.md#connect-versus-enroll) for configuration directory and credential handling.

### 8. Map each named Service to a local port and run the Runtime

Configure one local TCP target per authorized Service using the Runtime's `--map` option or `map =` entries in `agent.conf`. Use the Service's stable wire identifier and the actual local address and port on the customer host. Enrollment does not write those mapping lines. Follow [Agent enrollment](enroll.md) for the Runtime configuration requirements.

Run `thirp-agent` with the enrolled configuration and the local mappings. In the console, confirm that the Agent and intended Services are online. If they remain offline, check that the Runtime is running, has the enrolled configuration, and has a target configured for each Service before proceeding.

### 9. Request access in the console

On the customer Environment, use **Request access**. Select the required Services, enter a **Reason**, choose a bounded duration, and optionally supply the **Ticket or case reference**. Submit the request and open its request page. A request can contain several Services; approval creates a separate grant for each one. See [Access requests and grants](access.md).

### 10. Have the customer Approver approve in the browser

The customer Approver signs in, opens the request, checks its Services, requester, duration, and reason, and explicitly chooses **Approve**. Opening a request link only displays the request. An MCP call or a chat “yes” cannot approve it. If the customer denies the request, stop; otherwise wait until the relevant grant is active.

### 11. Connect from the vendor machine

Install the CLI appropriate for the vendor machine as described in [CLI installation](cli.md), and use `thirp login` to authenticate as the user whose grant will be used. Select the same Organization and Environment using the CLI options described in [CLI session](cli.md#session). For each authorized Service, open a local listener:

```text
thirp connect SERVICE --listen 127.0.0.1:PORT
```

Replace `SERVICE` with the named Service identifier and `PORT` with an available local port on the vendor machine. Point your support client at that listener. `thirp connect` uses an existing active grant; it does not create an access request. Run only your own authorized product checks during the approved window.

### 12. Revoke or let the window expire

When support finishes, an Approver or other authorized tenant role can use **Revoke grant** on each grant, or use `thirp access revoke GRANT`. A request covering multiple Services has multiple grants to end. Otherwise, each grant expires at the end of its window. Expiry or revocation resets live streams for that grant and denies new connections under it. See [Access: revoke](access.md#revoke).

## Agentic

### 1. Establish the same customer foundation

Start a trial and sign in at [the signup page](https://api.thirp.io/console/signup), create or select the vendor Organization, and establish the customer's Environment and active customer Approver through the existing invitation flow. Create the named Services in the console. The [Console steps 1–5](#1-start-a-trial-and-sign-in) describe this foundation, including the Environment's OS and architecture fields.

Supply OS and architecture from knowledge of the customer host. The vendor's AI client and the machine reading the install email are separate from that host. The customer-side Thirp Agent remains the enrolled `thirp-agent` running on the customer machine.

### 2. Point the vendor's AI client at MCP

Use `POST https://api.thirp.io/v1/mcp` with an Organization Service Identity or the person's signed-in Thirp session. Authenticate every request and keep credentials out of chat, URLs, logs, and skill files. Configure your AI client according to its own support for MCP; the endpoint alone does not establish a session or permission to act.

Follow [Agents and MCP](agents.md) for the ten tools, authentication, Streamable HTTP, JSON-RPC, and argument schemas. The [live OpenAPI](https://api.thirp.io/v1/openapi.yaml) is the corresponding API contract. MCP arranges access and retrieves its records; the customer still makes the approval decision in the browser.

### 3. Find or create the Environment and confirm its platform

The AI client can call `list_environments` to inspect visible Environments, their Services, `agent_status`, `os`, and `arch`. Use `ensure_environment` to select or create the named Environment with `name` and `region_id`, optionally supplying the customer and deployment references. See [Agents and MCP](agents.md) for the exact selection rules.

Creating or selecting an Environment without a platform is allowed. Before installer mail, supply both `os` and `arch` or confirm that both are already stored. `ensure_environment`, Environment create/PATCH, and `send_agent_install` can save the pair; omitting both keys reuses stored values. The console can also set them. Use only the four supported pairs and downloads in the [installation table](#6-install-the-matching-cli-and-runtime-on-the-customer-host).

If the Environment and call both lack a platform, `send_agent_install` refuses and sends no mail. Do not default to Linux or infer a platform from a browser or mail client. Obtain the customer host's OS and architecture, set both, and then send the mail. Partial, null, empty, and unsupported pairs refuse too.

### 4. Email the installer and wait for the customer-side Runtime

Call `send_agent_install` for the Environment with the customer's `recipient` and the required wire `service_ids`. Supply OS and architecture together if they are not already stored. The email includes the matching CLI and Runtime URLs, checksums, and one-use enrollment command; install mail does not assign approval authority.

The customer follows [Console steps 6–8](#6-install-the-matching-cli-and-runtime-on-the-customer-host): verify and install both downloads, run `thirp agent enroll`, configure each Service's local TCP target, and run `thirp-agent`. The AI client waits for the intended Agent and Services to be online. Emailing instructions alone does not bring the customer host online.

Install uses wire Service IDs; an access request uses catalog Service UUIDs. If additional Service authorization is needed for an existing Agent, follow the repeat-install procedure in [Agents and MCP](agents.md), including the same `display_name`, another enrollment exchange, and Runtime target configuration. There is no `ensure_services` tool.

### 5. Request access and email the customer review link

Call `request_access` with the Environment, required `catalog_service_ids`, a reason, and the shortest useful duration; include `external_case_id` when applicable. Then call `send_approve_link` with the resulting request ID. This emails the review link to the Environment's active Approvers. Establish an Approver with a verified email through the invitation flow first; installer mail does not do this.

The Approver opens the browser link, signs in, inspects the request, and explicitly approves on the page. A chat “yes” is never approval, and an MCP call cannot approve. Poll `grant_status`; stop on denial or expiry, and wait for each required grant to be active. Inspect individual grants when a multi-Service request has mixed states.

### 6. Connect and run the vendor's own checks

Call MCP `connect` for each active grant. It returns connection details and private bounded credentials for the existing CLI/SDK Broker path. MCP does not carry the TCP stream. Use the CLI or SDK to open that stream, as described in [Agents and MCP](agents.md) and [Access requests and grants](access.md). Keep returned Caller tokens private.

The vendor runs its own authorized checks against the connected Service and interprets the results. Thirp supplies the connection primitives. The [example diagnose skill](examples/diagnose-skill.md) is **EXAMPLE ONLY**, not a built-in Thirp diagnostic. There is no diagnose tool.

### 7. Retrieve the report

Call `get_report` with the request ID to retrieve the request, grants, connection history, and permitted audit facts. Summarize those records alongside the vendor's own observations and case reference. Audit facts require the existing audit-reader role; see [Agents and MCP](agents.md). Keep credentials out of the report.

### 8. Revoke every grant

Call `revoke_access` for each grant when the support work finishes, including when a vendor check fails. Confirm the resulting state with `grant_status` and close client streams. Revocation and expiry reset streams for the affected grant; subsequent `connect` calls refuse outside its window or after revocation. Fetch `get_report` again if you need the final connection and revocation records.
