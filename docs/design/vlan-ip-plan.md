# VLAN and IP Address Plan — assigns a VLAN ID, subnet, and addressing rules to each segment.

**Status:** Approved · **Version:** 1.0 · **Last updated:** 2026-08-06 · **Owner:** Ioannis Mintzivyris

## 2. Overview

This document gives each network segment a number and a block of addresses. It assigns a VLAN ID (a label that separates traffic on shared switch hardware), a subnet (the range of addresses that segment uses), a gateway (the address hosts send traffic to when leaving the segment), and rules for which addresses are fixed by hand and which are handed out automatically.

The segments themselves, and the reason each one exists, are defined in the Segmentation Design. This document does not repeat that reasoning — it puts numbers to it.

## 3. Scope

Covers VLAN IDs, subnets, gateway placement, and the split between fixed and automatic addressing for the three Phase 1 segments.

Does not cover which segments may talk to each other or any filtering rule (see Routing and ACL Design), nor the switch and router commands that apply these numbers (see Device Configuration and Bring-Up).

Covers Phase 1 only. Segments deferred to later phases are given room in the numbering scheme but are not addressed here.

## 4. Dependencies

- Segmentation Design (Approved 1.0) — the three segments this document numbers.
- Physical Port Map (Approved 1.2) — which device sits on which interface.

## 5. Deliverables

A table assigning every segment a VLAN ID, name, subnet, and gateway.

An addressing convention that applies inside every host segment — which address ranges are fixed by hand, which are handed out automatically, and which are reserved.

A defined DHCP range for the Servers segment, ready for the clients that arrive in Phase 3.

## 6. Detailed content

### 6.1 Addressing scheme

All internal addressing comes from the private **10.x** range. The 172.x range is deliberately avoided: the ISP's internal resolver sits at a 172.x address, and using that range internally could cause an address clash.

The **VLAN ID is echoed into the second number of the address.** VLAN 10 uses 10.**10**.x.x, VLAN 20 uses 10.**20**.x.x. Reading any address tells you its segment on sight.

VLAN IDs are spaced in tens (10, 20, 30) rather than counted (1, 2, 3). The gaps leave room for related segments to join later — the Phase 3 server-side segments can sit at 21, 22, 23, next to the Servers segment they belong with.

VLAN 1 is never used. It is the switch default, where every port lands until told otherwise, which makes it both a common target and a common source of mistakes.

### 6.2 The VLAN and subnet table

| VLAN | Name | Purpose | Subnet | Gateway |
|---|---|---|---|---|
| 10 | mgmt | Management — device logins, hardware controllers, PAW-02 | 10.10.0.0/24 | 10.10.0.1 |
| 20 | srv | Servers — the machines running lab services | 10.20.0.0/24 | 10.20.0.1 |
| 30 | transit | Point-to-point link, core01 ↔ rtr01 | 10.30.0.0/30 | 10.30.0.1 |

A **/24** is a subnet holding 254 usable addresses. A **/30** holds two — exactly enough for the two ends of a point-to-point link and nothing else.

The gateway is the **first usable address (.1)** in every segment. The gateway for the two host segments is a virtual interface on core01 (a switch interface that acts as the gateway for a VLAN). **This follows the settled decision that all routing lives on the core.**

For Transit, .1 is the core01 end and .2 is the rtr01 end.

### 6.3 Addressing convention inside a host segment

The same layout applies to every /24 host segment, so any address reads the same way across the network:

| Range | Use |
|---|---|
| .1 | Gateway |
| .2 – .99 | Fixed addresses, set by hand (servers, controllers, device logins) |
| .100 – .239 | Automatic range (DHCP), handed out to clients |
| .240 – .254 | Reserved for special devices — redundant pairs, shared-service addresses |

**DHCP** (a service that hands out addresses automatically) draws only from the .100–.239 block, so an automatic address can never collide with a fixed one. The .100 line is a readable boundary: below it is assigned by hand, at it and above is leased.

### 6.4 Fixed versus automatic addressing per segment

**Servers (VLAN 20)** — server addresses are fixed by hand. A server is an anchor: other machines find it and route to it by a known address, so that address must not move. The DHCP range is defined now but serves no clients until Phase 3, when virtual machines and workstations arrive. dc01 runs the DHCP service.

**Management (VLAN 10)** — all fixed, no automatic range. A hardware controller must answer at a known address when its server is down; a leased address that could change defeats the purpose. The segment keeps the same .1 / .2–.99 / .240+ shape for consistency, but no DHCP range is active on it.

**Transit (VLAN 30)** — two addresses, both fixed. No automatic addressing on a point-to-point link.

## 7. Acceptance criteria

- Every VLAN has an ID, a name, a purpose, and a subnet.
- Every host segment has a gateway address.
- The fixed, automatic, and reserved ranges are defined and do not overlap.
- The Servers segment has a defined DHCP range.
- No filtering rule or device command appears in this document.

## 8. References

- Segmentation Design — the segments this document numbers
- Physical Port Map — device-to-interface mapping
- Routing and ACL Design — which segments may talk, and the filtering rules *(not yet written)*
- Device Configuration and Bring-Up — the commands that apply these numbers *(not yet written)*
- Master Document — where this document sits in the repository