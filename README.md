# Secure Network Architecture Lab

A segmented, defense-in-depth network built and validated in a virtual lab. This
project demonstrates network zoning, least-privilege firewall policy, and
intrusion detection — and, more importantly, documents the *reasoning* behind
each design decision and proves the controls hold under attack.

> This is a design-and-validation exercise, not a tool walkthrough. The goal is
> to show how trust boundaries are chosen, enforced, and verified.

---

## 1. Objective

Design a small enterprise-style network that contains the blast radius of a
compromise. Specifically: if the public-facing service is breached, the attacker
must not be able to reach internal or management systems. The lab builds the
network, hardens it with least-privilege firewall rules, and then validates the
boundaries by attacking them.

**What this demonstrates to a reader:** network segmentation, default-deny
firewall design, threat modeling, and evidence-based verification.

---

## 2. Threat model

The design assumes the following, and is built to survive it:

- **Assume the DMZ host is compromised.** The public web server is the most
  exposed asset. The controlling question of this design is: *once an attacker
  owns the DMZ host, what can they reach?* The answer must be "nothing of value."
- **Assume an insider foothold is possible.** A device on the corporate LAN may
  be attacker-controlled (phishing, malicious USB). It should not be able to
  freely reach the DMZ or the management plane.
- **The management plane is the highest-value target.** Administrative access to
  the firewall is effectively root over the entire network, so access to it is
  the most tightly restricted boundary.

Out of scope for this lab: physical security, endpoint hardening, and
application-layer vulnerabilities in the DMZ host (the host is intentionally
vulnerable — it is the target, not the subject).

---

## 3. Architecture

<!-- TODO: export your draw.io topology to diagrams/topology.png and it will render here -->
![Network topology](diagrams/topology.png)

Four zones, each isolated, all routed through a single firewall chokepoint:

| Zone | Subnet | Purpose | Trust level |
|------|--------|---------|-------------|
| WAN | (NAT) | Simulated internet — untrusted | None |
| Management | 10.10.10.0/24 | Firewall administration | Highest restriction |
| Corporate LAN | 10.10.20.0/24 | Internal user workstations | Trusted |
| DMZ | 10.10.30.0/24 | Public-facing service (vulnerable target) | Untrusted-internal |

The firewall is the only path between any two zones. There is no direct
zone-to-zone connectivity.

---

## 4. Firewall policy (the core of the design)

Every rule below exists for a stated reason. Everything not explicitly allowed
is denied by the implicit default-deny at the bottom of each interface.

| Source | Destination | Action | Rationale |
|--------|-------------|--------|-----------|
| Corporate | WAN | Allow (HTTP/S, DNS) | Users need internet access |
| Corporate | DMZ | **Deny** | No business reason for users to reach the DMZ directly |
| Corporate | Management | **Deny** | Users must never touch the admin plane |
| DMZ | WAN | Allow (HTTP/S only) | Limited outbound for updates; nothing else |
| DMZ | Corporate | **Deny** | A compromised web server must not reach internal hosts |
| DMZ | Management | **Deny** | Contain the highest-risk zone away from admin |
| WAN | DMZ | Allow (80/443 to web server only) | The single reason the DMZ exists |
| WAN | Corporate | **Deny** | Internal hosts are never internet-facing |
| WAN | Management | **Deny** | Admin plane is never reachable from outside |
| Designated admin host | Management | Allow | Only one host may administer the firewall |

Per-interface rule details and screenshots live in [`firewall-rules/`](firewall-rules/).

---

## 5. Validation

The design is not "secure because I said so" — each boundary is tested.

### Before hardening (baseline)
<!-- TODO: screenshot a successful ping from Corporate to DMZ -->
With only the default interface rules, lateral movement between zones is
possible. See `evidence/before-segmentation.png`.

### After hardening
<!-- TODO: screenshot the same ping timing out -->
With least-privilege rules in place, the Corporate-to-DMZ path is blocked and
the DMZ cannot initiate contact with internal zones. See
`evidence/after-segmentation.png`.

### Attack simulation
<!-- TODO: screenshot the nmap scan and the resulting Suricata alert -->
An attacker (Kali) scanning the DMZ is detected by the IDS, and attempts to
pivot from a compromised DMZ host to the corporate zone are blocked at the
firewall. See `evidence/suricata-alert.png`.

---

## 6. Detection layer

Suricata (IDS) inspects traffic entering from the WAN and within the DMZ, using
the ET Open rule set. It runs in alert-only (IDS) mode for observability; inline
blocking (IPS) is noted as a follow-up once alert behavior is understood.

---

## 7. Lessons learned

See [`lessons-learned.md`](lessons-learned.md) for what surprised me, what I'd do
differently, and where the design has known limits.

---

## 8. Tooling

| Tool | Role |
|------|------|
| VirtualBox | Hypervisor hosting the lab |
| pfSense CE / OPNsense | Firewall and router |
| Kali Linux | Attacker |
| Metasploitable 2 | Vulnerable DMZ target |
| Suricata | Intrusion detection |
| draw.io | Topology diagram |

---

## 9. Possible extensions

- **Isolated OT/ICS zone** — an OpenPLC device on Modbus in its own segment,
  demonstrating why industrial protocols (which lack built-in authentication)
  require strict network isolation.
- **IDS → IPS** — moving Suricata from alert-only to inline blocking.
- **Centralized logging** — forwarding firewall and IDS logs to a single
  collector for correlation.
