# WAN interface rules

The WAN is the untrusted internet side. The only inbound traffic permitted is
to the single published DMZ service. Everything else is denied.

| # | Action | Source | Destination | Port | Rationale |
|---|--------|--------|-------------|------|-----------|
| 1 | Pass | any | DMZ web server IP | 80/443 | Publish the one intended service |
| 2 | (implicit) Deny | any | any | any | Default-deny catches everything else |

<!-- TODO: add a screenshot of the WAN rule list from the firewall GUI -->
