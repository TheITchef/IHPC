# Physical Port Map — defines every cabled connection in the lab and where it lands.

**Status:** Draft · **Version:** 0.1 · **Last updated:** 2026-07-27 · **Owner:** Ioannis Mintzivyris

## 2. Overview

This document records the physical network connections of the lab: which device interface connects to which patch panel port, and which switch port that patch panel port lands on. It is the authoritative record of what is physically cabled to what. A person holding this document should be able to trace any link end to end without touching the rack.

Device hostnames used here are target names. The devices are at factory default and do not yet carry them; they are applied during device configuration and bring-up.

## 3. Scope

Covers physical layer connectivity only — device interfaces, patch panel mapping, switch ports, and direct runs that bypass the patch panel. Does not cover logical configuration such as VLANs, IP addressing, or routing. Does not cover interface hardware identity such as MAC addresses.

## 4. Dependencies

None. This document records observed physical state and depends on no prior document.

## 5. Deliverables

A patch panel mapping table, a port table for each network device, a record of direct runs bypassing the patch panel, and a statement of unused and reserved capacity.

## 6. Detailed content

### 6.1 Baseline

All devices were newly procured and racked, then wiped to factory default before Phase 1 began. There is no prior configuration state to preserve, and the rollback target for all Phase 1 work is factory default.

### 6.2 Patch panel mapping (24-port)

Read each row as a single physical run: the device interface on the front side, the switch port it lands on via the rear.

| PP | Device side | Switch side |
|---|---|---|
| 01 | itc-uvy-esxi01 LOM1 | itc-uvy-core01 Gi1/0/1 |
| 02 | itc-uvy-esxi01 LOM2 | itc-uvy-core01 Gi1/0/2 |
| 03 | itc-uvy-ms01 LOM1 | itc-uvy-core01 Gi1/0/3 |
| 04 | itc-uvy-ms01 LOM2 | itc-uvy-core01 Gi1/0/4 |
| 05–09 | *unused* | *unused* |
| 10 | itc-uvy-dc01 LOM1 | itc-uvy-core01 Gi1/0/10 |
| 11–12 | *unused* | *unused* |
| 13 | itc-uvy-esxi01 iDRAC | itc-uvy-oob01 Gi0/1 |
| 14 | itc-uvy-esxi02 iDRAC | itc-uvy-oob01 Gi0/2 |
| 15 | itc-uvy-ms01 iLO | itc-uvy-oob01 Gi0/3 |
| 16–17 | *unused* | *unused* |
| 18 | itc-uvy-oob01 Gi0/9 (uplink, rear-facing) | itc-uvy-core01 Gi1/0/48 |
| 19–20 | *unused* | *unused* |
| 21 | itc-uvy-rtr01 Gi7 | itc-uvy-core01 Gi1/0/46 |
| 22 | *unused* | *unused* |
| 23 | *spare — nothing connected* | itc-uvy-core01 Gi1/0/23 |
| 24 | *spare — nothing connected* | itc-uvy-core01 Gi1/0/24 |

Ports 23 and 24 are patched on the switch side only. They are prepared drops awaiting a device.

### 6.3 Core switch — itc-uvy-core01 (Cisco WS-C3850-48T)

The core switch carries all production traffic and is the Layer 3 boundary for the lab. All inter-VLAN routing happens here.

| Port | Connects to |
|---|---|
| Gi1/0/1 | PP01 → itc-uvy-esxi01 LOM1 |
| Gi1/0/2 | PP02 → itc-uvy-esxi01 LOM2 |
| Gi1/0/3 | PP03 → itc-uvy-ms01 LOM1 |
| Gi1/0/4 | PP04 → itc-uvy-ms01 LOM2 |
| Gi1/0/10 | PP10 → itc-uvy-dc01 LOM1 |
| Gi1/0/23 | PP23 — prepared drop, no device |
| Gi1/0/24 | PP24 — prepared drop, no device |
| Gi1/0/46 | PP21 → itc-uvy-rtr01 Gi7 |
| Gi1/0/48 | PP18 → itc-uvy-oob01 Gi0/9 (management uplink) |
| all other Gi1/0/x | *unused* |
| Te1/1/1–Te1/1/4 | *no uplink module fitted* |

The uplink module bay is empty. The switch has no 10G uplink capacity in its current configuration.

### 6.4 Management switch — itc-uvy-oob01 (Cisco WS-C3560CG-8PC)

It exists so that baseboard management controllers (BMCs) — iDRAC on Dell hardware, iLO on HPE — remain reachable when the production path is unavailable.

| Port | Connects to |
|---|---|
| Gi0/1 | PP13 → itc-uvy-esxi01 iDRAC |
| Gi0/2 | PP14 → itc-uvy-esxi02 iDRAC |
| Gi0/3 | PP15 → itc-uvy-ms01 iLO |
| Gi0/4–Gi0/7 | *unused* |
| Gi0/8 | PAW-01 (ASUS PN52) — disconnected for the Phase 1 build |
| Gi0/9 | PP18 → itc-uvy-core01 Gi1/0/48 (uplink to core) |
| Gi0/10 | *unused* |

PAW-01 is a general-purpose workstation and is therefore not a privileged access workstation. It is disconnected from the management segment for the duration of the Phase 1 build; the build is performed over console.

### 6.5 Edge router — itc-uvy-rtr01 (Cisco 891F)

The 891F has one dedicated WAN interface (Gi8) and eight switched LAN interfaces (Gi0–Gi7) sharing a single Layer 2 domain.

| Interface | Connects to |
|---|---|
| Gi0–Gi6 | *unused* |
| Gi7 | PP21 → itc-uvy-core01 Gi1/0/46 |
| Gi8 (WAN) | Home router — direct run |

### 6.6 Direct runs (bypassing the patch panel)

| Run | Purpose | Status |
|---|---|---|
| itc-uvy-rtr01 Gi8 → home router | WAN uplink | Permanent. Normal for an edge run. |
| itc-uvy-dc01 LOM2 → home router | Temporary build lifeline for operating system installation and updates | **Temporary.** Removed once the core switch is live and dc01 reaches the internet via itc-uvy-rtr01. |

Until removed, the dc01 lifeline is a path that bypasses the segmentation design entirely. Its removal is a task in Validation and Handover, not an intention.

A third direct run, core01 Gi1/0/26 → itc-uvy-ms01 LOM4, existed as a troubleshooting leftover from a previous lab iteration. It has been disconnected and is not part of this map.

### 6.7 Unused and reserved capacity

| Device | Interfaces | Cabled | Free |
|---|---|---|---|
| itc-uvy-dc01 (T330) | 2 | LOM1 (patched), LOM2 (lifeline) | 0 |
| itc-uvy-esxi01 (R620) | 8 — 4 onboard + 4 PCIe | LOM1, LOM2 | 6 |
| itc-uvy-esxi02 (R620) | 4 | none — iDRAC only | 4 |
| itc-uvy-ms01 (DL360) | 4 | LOM1, LOM2 | 2 |



itc-uvy-esxi02 carries four onboard interfaces only, with no PCIe card fitted. The asset inventory is correct for this device.

itc-uvy-esxi02 has no data interfaces cabled. Its role remains undecided and will be recorded in an Architectural Decision Record when Phase 3 begins.

Patch panel ports 05–09, 11, 12, 16, 17, 19, 20 and 22 are unused on both sides.

## 7. Acceptance criteria

- Every cabled link appears in exactly one table.
- Every link is traceable end to end from this document alone, without access to the rack.
- Every direct run is recorded with a purpose, and every temporary run with a removal condition.
- The greenfield baseline is stated.
- Unused capacity is stated per device.

## 8. References

- Master Document — where this document sits in the repository map
