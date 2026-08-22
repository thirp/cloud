# Customer Approver

An Environment-scoped Approver decides access for that Environment. The Approver can approve, deny, and revoke.

The Approver sees the assigned Environment: its Services, access requests, and grants. The Approver does not see other customer names, other Environments, or Organization-wide membership as a side effect of approval.

## Decide

A pending access request names the User, Service, duration, and reason. Approve starts a grant. Deny leaves no grant.

Revoke ends an Active grant. Live streams for that grant RESET. Expiry does the same when the window ends.

## Console and API

The browser console is `https://api.thirp.net/console`, HTML over the public `/v1` API, not a second contract. Approver actions are the same access-request and grant operations the API exposes.

Transactional mail may notify the Approver of a new request and the requester of a decision. Invitation links do not contain tokens in the URL.
