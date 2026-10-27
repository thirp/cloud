# Publishing a log file as a Service

The Agent dials a local TCP target. A log file is not one. If the product only writes a file and you do not want to change it, run a loopback listener that, on connect, tails the file. `socat` is the usual helper. It is not part of Thirp.

```bash
socat TCP-LISTEN:9100,bind=127.0.0.1,reuseaddr,fork \
  EXEC:"tail -n 200 -F /var/log/vendor/app.log"
```

`-n 200` is a short historical buffer. `-F` follows across rotation. `fork` starts a new `tail` per connect so a second grant is not a one-shot. Bind loopback only; this is not a customer inbound port.

Map that port to the catalog Service the Agent is allowed to register. Enroll does not write `map=` lines; local targets are operator-side.

```bash
thirp-agent --map app-debug-log=127.0.0.1:9100
```

The same line in `agent.conf` is `map = app-debug-log=127.0.0.1:9100`. Access is still a grant: `thirp access request app-debug-log`, then `thirp connect`.

This is not a log shipper. Do not point Fluent Bit, Vector, or Filebeat at the vendor. The sidecar should speak when the Agent dials, for the grant window.
