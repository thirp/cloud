# Privacy and metadata

Thirp Cloud stores identity, tenancy, catalog, grants, audit metadata, and usage counters. It does not store application payloads.

## Stored

- Organizations, Users, memberships, Environments, invitations, sessions (hashed tokens)
- Services, Agents, machine identities, credential hashes and expiry
- Access requests and grants, Caller credential hashes
- Audit events (tenant-visible; retained 90 days)
- Usage: connection counts, duration, bytes, concurrency
- Organization events and webhook delivery attempts
- Whether a Service is currently registered (lease-based live status)

## Not stored

- Application stream contents
- Plaintext credentials after issuance (hashes only; plaintext returned once)
- Invitation tokens in console links (the link uses an id, not the token)

## Contacts

Privacy questions: `privacy@thirp.net`
