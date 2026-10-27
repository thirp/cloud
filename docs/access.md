# Access requests and grants

Access to a named Service is explicit. A User requests a bounded grant. The customer controls the Environment approval policy: they may require a customer Approver decision on every request, or expressly delegate bounded approval authority to the vendor with audit evidence and revocation control. Only after an authorized decision can `thirp connect` open a stream.

```text
thirp access request SERVICE [--service SERVICE] --duration DURATION --reason TEXT
customer Approver approves
  (or vendor Owner/Administrator approves only under active customer delegation)
thirp connect SERVICE --listen 127.0.0.1:PORT
```

`thirp connect` does not create an access request.

## Request

`thirp access request` records one or more Services in the same Environment, a shared duration, and a reason. Size 1 is `SERVICE` alone. Repeatable `--service` (and extra positionals before flags) add more catalog Services. Optional `--case` attaches an external case id. Create uses `catalog_service_ids`.

Amendment: earlier docs described one Service per request because that was the shipped row. Grant, connect, and revoke stay one Service.

`thirp access status` reports a request or grant.

## Grant

An approved request becomes one AccessGrant per listed Service, each with the same start and expiry. Each grant is scoped to that User and exact Service.

While the grant is Active, `thirp connect` may issue a short-lived Caller credential and open streams. After expiry, new CONNECT is denied. Live streams for that grant RESET.

## Revoke

An Approver (or other authorized tenant role) can revoke a grant. Existing streams for that grant RESET. Other grants stay up. A new CONNECT with the old Caller credential is denied.

`thirp access revoke GRANT` is the CLI path.

## Observable failure

These messages are what the CLI reports. They are tenant-visible, not operator diagnostics.

```text
not authenticated
not authorized
approval required
request pending
request denied
grant not yet active
grant expired
grant revoked
Service disabled
Service offline/not registered
Agent unavailable
Broker unavailable
```
