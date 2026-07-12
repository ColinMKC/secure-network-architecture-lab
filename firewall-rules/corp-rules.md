# Corporate LAN interface rules

Internal users may reach the internet but not the DMZ or the management plane.
Rule order matters: specific blocks sit above the general internet-allow.

| # | Action | Source | Destination | Port | Rationale |
|---|--------|--------|-------------|------|-----------|
| 1 | Block | Corporate net | DMZ net | any | No reason for users to reach the DMZ |
| 2 | Block | Corporate net | Management net | any | Users must never touch the admin plane |
| 3 | Pass | Corporate net | any | 80/443, 53 | Internet + DNS for normal work |
| 4 | (implicit) Deny | any | any | any | Default-deny |

<!-- TODO: add a screenshot of the Corporate rule list from the firewall GUI -->
