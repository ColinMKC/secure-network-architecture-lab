# DMZ interface rules

The DMZ is treated as hostile. It may reach the internet for updates but must
never initiate contact with internal or management zones. This is the boundary
that contains a web-server compromise.

| # | Action | Source | Destination | Port | Rationale |
|---|--------|--------|-------------|------|-----------|
| 1 | Block | DMZ net | Corporate net | any | Contain a compromised host |
| 2 | Block | DMZ net | Management net | any | Keep the admin plane unreachable |
| 3 | Pass | DMZ net | any | 80/443 | Limited outbound for updates only |
| 4 | (implicit) Deny | any | any | any | Default-deny |

<!-- TODO: add a screenshot of the DMZ rule list from the firewall GUI -->
