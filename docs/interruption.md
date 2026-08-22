# Interruption behavior

A Region has one Active Broker behind a stable public endpoint. When that Broker is replaced or becomes unavailable, **existing streams are not resumed**.

Tenant-visible Region `operational_status`:

```text
healthy
draining
replacing
unavailable
```

During drain or replace, new connections may be refused. Agents re-register when the next Broker generation is Active. Callers open new connections. There is no stream migration and no automatic cross-region failover.

Grant revoke and grant expiry RESET only that grant's streams. Other grants stay up.

Application payloads are not stored by the control plane or inspected by the Broker. A Broker restart does not replay bytes.
