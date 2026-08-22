# Security

Report vulnerabilities in Thirp Cloud and the `thirp` CLI to `security@thirp.net`.

Do not open a public GitHub issue for an unfixed vulnerability.

`thirp/cloud` is a distribution surface. It does not accept public documentation pull requests.

- Product and design-partner questions: `partners@thirp.net`
- Privacy: `privacy@thirp.net`

`security@thirp.net` is not the pilot-intake mailbox.

## Release signing

CLI release `SHA256SUMS` files may include a detached signature `SHA256SUMS.asc`. The Thirp publish key fingerprint is:

```text
3B8559D8754FB3C5B21110C786897A405CF3D8C4
```

Verify with `gpg --verify SHA256SUMS.asc SHA256SUMS`. Unsigned checksums are still usable; the signature is additional.
