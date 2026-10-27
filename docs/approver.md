# Customer Approver

An Environment-scoped Approver decides access for that Environment and controls that Environment's approval policy. The Approver can require customer approval for every request, or expressly permit the vendor to approve bounded requests with audit evidence. The Approver can approve, deny, revoke grants, and revoke a vendor-approval delegation.

The Approver sees the assigned Environment: its Services, access requests, grants, and current approval policy. The Approver does not see other customer names, other Environments, or Organization-wide membership as a side effect of approval.

A vendor Owner or Administrator cannot approve unless the customer has granted an active delegation for that Environment. Role alone is not enough.

## Decide

A pending access request names the User, the listed Service(s), duration, and reason. Approve starts a grant for each listed Service. Deny leaves no grants.

Amendment: earlier copy said “Approve starts a grant” for a single Service because that was the shipped request row.

Revoke ends an Active grant. Live streams for that grant RESET. Expiry does the same when the window ends.

## Console and API

The browser console is `https://api.thirp.io/console`, HTML over the public `/v1` API, not a second contract. Approver actions are the same access-request and grant operations the API exposes.

Transactional mail may notify the Approver of a new request and the requester of a decision. Invitation links do not contain tokens in the URL.
