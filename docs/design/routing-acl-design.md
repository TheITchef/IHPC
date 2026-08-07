# Routing and ACL Design — defines where routing happens and which flows are permitted across the lab.

**Status:** Draft · **Version:** 0.1 · **Last updated:** 2026-08-07 · **Owner:** Ioannis Mintzivyris

## 2. Overview

This document records two settled decisions and justifies them: **where routing happens**, and **which traffic is allowed to cross a boundary**.

Routing lives on the core. The edge does only what a WAN edge must — it faces the internet and translates addresses. Nothing else routes.

Filtering follows one principle: **default-deny**. Nothing is permitted unless it is written down here with a reason. This applies most strictly to two boundaries — the **management segment**, which nothing may reach except from the single trusted workstation, and the **edge**, which nothing from the internet may enter. Every permitted flow is an explicit, reasoned exception to the deny baseline.

The document does not list device commands (see Device Configuration and Bring-Up) — it defines the intent those commands will implement.

## 3. Scope

Covers:

- **Routing responsibilities** — which device routes what, the transit link between them, and the return routes at the edge.
- **The management flow matrix** — every flow permitted into the management segment, with a written reason for each.
- **The edge posture** — what may enter from the internet.

Does not cover:

- The switch and router commands that implement these decisions (see Device Configuration and Bring-Up).
- Internal filtering policy for the Servers segment beyond its default-deny relationship with management. Server-side flows are defined when servers are configured.
- **Remote administration.** In Phase 1 all administration is local. Remote access is a deferred decision, pending a tooling choice. An Azure tenant already exists and may host the eventual path (VPN or Bastion the likely candidates), but that use is not yet confirmed.
- **In-band SSH to network devices.** In Phase 1 the network devices are administered by serial console. In-band SSH is a later expansion, not enabled here.
- **DNS and time (NTP) for management.** Deferred to Phase 2, when an internal resolver and time source exist.

Covers Phase 1 only.

## 4. Dependencies

- **VLAN and IP Address Plan** (Approved 1.0) — the segments, subnets, gateways, and addressing convention this document filters and routes.
- **Segmentation Design** (Approved 1.0) — why each segment exists and the trust relationships between them.
- **Physical Port Map** (Approved 1.2) — which device sits on which interface, including the core–edge transit wiring and the WAN uplink.

## 5. Deliverables

- A **routing responsibility statement** — what the core routes, what the edge routes, the transit link between them, and the static return routes at the edge.
- A **management flow matrix** — the permitted flows into the management segment, each with a source, destination, service, direction, and written reason. Everything not listed is denied.
- An **edge posture statement** — what may enter from the internet, and why the inbound deny is load-bearing.
- A **deferred-decisions register** — the flows and choices consciously left for a later phase (remote administration, in-band SSH, management DNS and time), so each is recorded as chosen rather than missing.

## 6. Detailed content

### 6.1 Routing design

All routing between segments happens on the **core** (core01). Each host segment's gateway is a virtual interface on the core, so traffic moving from one VLAN to another is routed there and nowhere else. This is the settled decision from the VLAN and IP Address Plan: routing lives on the core.

The **edge** (rtr01) does only what a WAN edge must. It faces the internet, and it translates internal addresses to the public address for outbound traffic (**NAT** — Network Address Translation, the process of rewriting an internal address to a public one so internal hosts can be reached by return traffic). It also runs a **basic stateful firewall** — the control that decides what may cross the edge in each direction. The edge does not route between internal segments; it has no reason to see internal-to-internal traffic at all.

The two are joined by the **transit link** — a dedicated point-to-point subnet (VLAN 30, 10.30.0.0/30) with the core at .1 and the edge at .2. All traffic leaving the lab for the internet crosses this link from core to edge; all return traffic crosses back.

Because the edge does not participate in internal routing, it must be told how to return traffic to the internal segments. This is done with **static return routes** on the edge: fixed entries that say "to reach a lab subnet, send it back to the core across the transit link." Without them, return traffic would reach the edge and have no path home.

**Why this split:** the core is the single routing authority, so inter-segment policy is enforced in one place. The edge stays deliberately simple — a smaller, internet-facing device with the least configuration it can have, because everything exposed to the internet is a thing that must be defended.

### 6.2 Management posture

Management (VLAN 10) holds the controls of the estate — the device logins, the hardware controllers, and the single workstation permitted to use them. Because it controls everything else, it is treated as the most trusted tier: a **Tier 0** segment. Control flows one way — *from* management *to* the estate, never the reverse.

This inverts the normal posture. Most segments are permissive inward and filtered outward. Management is the opposite: **default-deny in both directions**. Nothing reaches a management interface, and management initiates nothing, unless a specific flow is written down with a reason.

Access to management comes from one place only: **PAW-02**, the privileged access workstation. An administrator does not reach a management interface from the general network — they work from PAW-02, which sits inside the segment as its sole trusted origin. Every permitted inbound flow in the matrix that follows begins at PAW-02.

This is why the segment is default-deny rather than simply firewalled: the goal is not to filter management traffic but to ensure management has almost no reachable surface at all. A compromise elsewhere in the estate finds nothing to talk to.

**One honest note for Phase 1:** PAW-02 is not yet hardened — in this phase it is an ordinary workstation acting as the trusted origin. This is stated openly rather than hidden. Hardening PAW-02 to full privileged-workstation standard is planned work in its own right; until it is done, the posture is correct by design, with end-to-end enforcement completed when PAW-02 is built out.

### 6.3 The flow matrix

The matrix is the record of every flow permitted into the management segment. **Default-deny is the baseline**: anything not listed here is dropped. Each row is an explicit exception, and each exception has a reason.

**Live flows (Phase 1):**

| # | Source | Destination | Service | Direction | Reason |
|---|---|---|---|---|---|
| 1 | PAW-02 | Hardware controllers (iDRAC, iLO) | HTTPS (tcp/443) | Into mgmt | Web administration of the hardware controllers, from the trusted origin only |
| 2 | PAW-02 | Managed devices | ICMP echo | Into mgmt | First-line reachability testing, from the trusted origin only |

Both live flows originate at PAW-02. No other source may initiate into management. This is the whole in-band management surface for Phase 1 — deliberately small.

**Note on dc01:** dc01 sits on the Servers segment (VLAN 20), not on management, so it is not a target in this matrix. It has iDRAC Express only — no remote console and no true out-of-band presence — and is administered locally by KVM. Its recovery path is physical.

**Deferred flows (not active in Phase 1):**

| Flow | Status | Reason for deferral |
|---|---|---|
| CLI to network devices (core01, rtr01) | Serial console, out-of-band | Network devices are administered by serial in Phase 1; in-band SSH is a later expansion |
| DNS and time (NTP) for management | Deferred to Phase 2 | No internal resolver or time source until dc01 is promoted (dc01 → DNS/DHCP; DC02 planned on ms01 for redundancy) |
| Remote administration into the lab | Deferred, tooling pending | All Phase 1 administration is local; remote path unconfirmed (Azure tenant exists; VPN or Bastion candidates) |

A deferred flow is a decision recorded, not a flow in force. None of the rows in the lower table permit any traffic today.

### 6.4 Edge posture

The edge faces the internet directly. rtr01's WAN interface (Gi8) receives a **public address by DHCP from the ISP** (from the 85.228.40.0/21 range), and there is **no NAT device in front of it** — the lab and the home network are peers on an unmanaged distribution switch, neither behind the other. The WAN interface is therefore reachable from the whole internet.

This makes the inbound deny **load-bearing, not theoretical**. On a typical home connection the ISP's own NAT hides everything behind it; here nothing hides the edge. Whatever the edge permits inbound is exposed to the open internet.

The posture is therefore **deny all inbound**. The stateful firewall on the edge permits only **return traffic for connections the lab itself started** — a session initiated from inside is allowed back, because the firewall remembers it. Nothing may *initiate* into the lab from the internet.

Two rules follow from this and are absolute for Phase 1:

- **Nothing internal is ever pinned to the WAN address.** No internal service, route, or dependency is tied to the public address — it is DHCP-assigned and can change, and it is internet-facing. It exists only as the lab's way out.
- **Remote administration is not solved by opening the edge.** No inbound port is opened for convenience — not SSH, not anything. Remote access, when it comes, arrives through a controlled channel decided separately (see the deferred flows in 6.3), never by making a hole in this deny.

**Known limitation, stated openly:** the edge firewall is the 891F's built-in stateful firewall — a branch router doing firewall duty, not a dedicated security appliance. This is adequate for the Phase 1 posture. A dedicated firewall is under consideration as a near-term improvement.

### 6.5 Segment relationship: management and servers

Management (VLAN 10) and Servers (VLAN 20) do not talk to each other in Phase 1, in either direction.

This falls out of the posture already defined. The management segment is default-deny inbound, so the Servers segment cannot reach it — no separate rule is needed; the deny already covers it. In the other direction, management has nothing it needs to reach on Servers in Phase 1, so nothing is permitted.

When Phase 2 introduces internal services — dc01 providing DNS, for example — the specific flow management needs will be added to the matrix as an **explicit, reasoned exception**. The two segments are opened to each other one named flow at a time, never as a general trust between VLANs.

## 7. Acceptance criteria

- Every live flow in the matrix has a source, a destination, a service, a direction, and a written reason.
- Default-deny is stated as the baseline for the management segment, with permitted flows shown as explicit exceptions.
- Default-deny inbound is stated for the edge, and the reason it is load-bearing — the directly-reachable public WAN address — is recorded.
- Routing responsibilities are written down: what the core routes, what the edge routes, the transit link between them, and the static return routes at the edge.
- Deferred flows and decisions are recorded as consciously deferred, distinguishable from live flows, so none reads as an omission.
- No device commands appear in this document.

## 8. References

- **VLAN and IP Address Plan** — the segments, subnets, gateways, and addressing convention this document routes and filters.
- **Segmentation Design** — why each segment exists and the trust relationships between them.
- **Physical Port Map** — device-to-interface mapping, including the core–edge transit wiring and the WAN uplink.
- **Device Configuration and Bring-Up** — the switch and router commands that implement the routing and filtering defined here *(not yet written)*.
- **Master Document** — where this document sits in the repository.