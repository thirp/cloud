# Thirp Cloud

This repository contains public documentation, API contracts, and downloadable client releases for Thirp Cloud. The Thirp Cloud service implementation is proprietary. The open-source data plane is Thirp Runtime at [github.com/thirp/runtime](https://github.com/thirp/runtime).

Thirp Runtime is the open-source named-Service data plane. Thirp Cloud adds managed identity, customer approval, timed grants, audit, usage, and hosted operation.

## CLI

The public command is `thirp`. First release archive: `thirp-cli-0.1.0-linux-amd64`. Windows and macOS archives are reserved for a later release; they are not shipped here.

Implemented commands:

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

`thirp connect` does not create an access request. Request access, wait for an Approver, then connect.

`thirp agent enroll` writes `agent.conf` and `agent.token` for the Runtime `thirp-agent` process. It does not replace `thirp-agent`.

`thirp-connect` is the standalone Runtime Caller with explicit local configuration. It is not this CLI.

Install and usage: [docs/cli.md](docs/cli.md).

## API

Public contract: [openapi/v1.yaml](openapi/v1.yaml), also served at `GET /v1/openapi.yaml` on `https://api.thirp.net`. It covers `/v1`, `/healthz`, `/readyz`. The console is `https://api.thirp.net/console`, an HTML presentation over that API, not a second contract.

## Docs

- [CLI install and usage](docs/cli.md)
- [Access requests and grants](docs/access.md)
- [Customer Approver](docs/approver.md)
- [Agent enrollment](docs/enroll.md)
- [Interruption behavior](docs/interruption.md)
- [Privacy and metadata](docs/privacy.md)
- [Webhooks](docs/webhooks.md)
- [Compatibility](docs/COMPATIBILITY.md)
- [Changelog](docs/CHANGELOG.md)
- [Security](docs/SECURITY.md)
- [Trademarks](docs/TRADEMARKS.md)
- [CLI releases](releases/README.md)

## License

Documentation and OpenAPI are CC-BY-4.0. CLI binaries are governed by [TERMS](TERMS). See [LICENSE](LICENSE) and [NOTICE](NOTICE).

Vulnerabilities: `security@thirp.net` ([docs/SECURITY.md](docs/SECURITY.md)). Product questions: `partners@thirp.net`. Privacy: `privacy@thirp.net`.
