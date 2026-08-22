# Access requests and grants

Access to a named Service is explicit. A User requests a bounded grant. A customer Approver decides. Only then can `thirp connect` open a stream.

```text
thirp access request SERVICE --duration DURATION --reason TEXT
customer Approver approves
thirp connect SERVICE --listen 127.0.0.1:PORT
```

`thirp connect` does not create an access request.

## Request

`thirp access request` records the Service, duration, and reason. Optional `--case` attaches an external case id.

`thirp access status` reports a request or grant.

## Grant

An approved request becomes an AccessGrant with a start and expiry. The grant is scoped to that User and exact Service.

While the grant is Active, `thirp connect` may issue a short-lived Caller credential and open streams. After expiry, new CONNECT is denied. Live streams for that grant RESET.

## Revoke

An Approver (or other authorized tenant role) can revoke a grant. Existing streams for that grant RESET. Other grants stay up. A new CONNECT with the old Caller credential is denied.

`thirp access revoke GRANT` is the CLI path.

## Observable failure

These messages are what the CLI reports. They are tenant-visible, not operator diagnostics.

```text
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
