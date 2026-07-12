# Lessons learned

Fill this in as you build — it is one of the most valuable parts of the repo,
because it shows reflection rather than rote execution.

- Unattended-install had to be disabled; ISO attached via Storage instead
- Had to eject the ISO after install to avoid the installer loop
- GUI came up on HTTP instead of HTTPS — diagnosed via ping (worked) + curl (found port 80)
- Unchecked "Block RFC1918" on WAN because the lab's WAN is a private NAT range
- webConfigurator defaulted to HTTP after console setup; switched to HTTPS so admin credentials aren't sent in plaintext.
- "OPT interfaces (CORP/DMZ) start with no rules = implicit default-deny; all traffic blocked including outbound, unlike LAN's default allow-all."

## What surprised me
<!-- e.g. the default rules were more permissive than expected on new interfaces -->

## What I would do differently
<!-- e.g. plan the addressing scheme before creating interfaces -->

## Known limitations of this design
<!-- e.g. no application-layer inspection; IDS in alert-only mode -->

## What I would add for a production version
<!-- e.g. centralized logging, redundant firewall, IPS mode, mTLS to management -->
