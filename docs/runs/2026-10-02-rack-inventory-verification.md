# Rack Inventory Verification Run — records the first hardware survey of the lab's servers, read at the rack.

**Status:** Approved · **Version:** 1.0 · **Last updated:** 2026-10-08 · **Owner:** Ioannis Mintzivyris

## 2. Overview

This document records a bench session, not a design. On 2026-10-02 the four servers in the rack were surveyed at the local console. The purpose was to replace assumed hardware facts with facts read from the hardware.

The session was read-only. Results are recorded as found. Where something was not checked, this document says so and gives the reason.

The session showed that the previous inventory sheet was wrong for three of the four servers. It also found that no server has disk redundancy, and that two baseboard management controllers (BMCs — the built-in chips that let a server be managed even when it is switched off) carry leftover configuration.

The session ended after the servers. Network devices, passive equipment, spares and endpoints were not checked.

## 3. Scope

### In scope

- The four servers: itc-uvy-dc01, itc-uvy-esxi01, itc-uvy-esxi02, itc-uvy-ms01. Checked.
- The network devices, the passive equipment (patch panel and power distribution units), spare disks, console cables, PAW-01 and the test host. Planned, not checked (see 6.2).

### Out of scope

- Any change to hardware or configuration.
- MAC addresses. They belong to interface identity, not to this record.
- Cabling, except the dc01 LOM2 check.
- IOS versions (the operating system on the Cisco devices). Reading them needs a console session, which is privileged access.

## 4. Dependencies

- **Physical Port Map** (Approved 1.2) — the interface and cabling claims checked here.
- **Segmentation Design** (Approved 1.0) — places every BMC in the management segment. Findings 2 and 3 are measured against it.
- **VLAN and IP Address Plan** (Approved 1.0) — the management subnet.
- **Device Configuration and Bring-Up** (Approved 1.2) — holds the server and BMC switch ports shut. Findings 2 and 3 rely on it. Port state was not checked in this session.

## 5. Deliverables

- **As-found results** — every checked item, with its evidence source.
- **Findings** — results that require a change, each with its evidence.
- **Deviations** — where the session departed from the plan or process.
- **Observations** — facts worth keeping that require no change.
- **End state** — the condition the hardware was left in.

## 6. Detailed content

### 6.1 Session setup and method

- **Date:** 2026-10-02, afternoon to evening, at the rack. Start and end times were not recorded.
- **Baseline:** all servers are treated as factory default, with no operating system and no role (owner decision).
- **Dell servers:** hardware inventory exported from the Lifecycle Controller (Dell's built-in setup and inventory tool) to a USB stick, as XML (a structured text file). Order: dc01, esxi01, esxi02.
- **HPE server:** read on screen in System Utilities (HPE's built-in setup screens) and Smart Storage Administrator (HPE's disk controller tool), and photographed.
- **Labels:** read and photographed where a screen could not show the value.
- **No network:** nothing was connected to a lab port. No BMC was reached over the network. All reading was at the local console.
- **Disk rule:** disk presence is taken from controller reports only. Carriers and labels are not evidence.
- **Raw evidence:** the exports and photos are held by the owner, outside the repository. They contain identifiers that do not belong in a public record.
- **Addresses left out:** no MAC address is reproduced here, nor the iLO's IPv6 address, which is derived from a MAC address.

### 6.2 Results

Sources of claims: **O** owner statement, 2026-10-02 · **I** previous inventory sheet, 2026-07-03, not in the repository · **P** Physical Port Map 1.2 · **—** no prior claim.

Evidence for results: **E** Lifecycle Controller export · **S** photographed screen · **L** photographed label · **R** owner reading at the rack.

Terms used in the tables: **LOM** — a network port built into the server board. **RAID1** — two disks holding identical copies. **RAID0** — data spread across disks with no copy; on a single disk it gives no protection. **Virtual disk / logical drive** — the disk the RAID controller presents to the server. **Backplane** — the board the drive bays plug into. **iDRAC** (Dell) and **iLO** (HPE) — the BMC types.

Serial numbers and service tags show their last four characters only. The full values are held with the raw evidence.

#### itc-uvy-dc01

| Field | Claimed | As found | Evidence | Result |
|---|---|---|---|---|
| Model | Dell PowerEdge T330 (I) | Dell PowerEdge T330 | E | Confirmed |
| Service tag | — | ***MYQ2 | E | Recorded |
| CPU | 1× Xeon E3-1220 v6 (I) | 1× Xeon E3-1220 v6, 4 cores, 4 threads | E | Confirmed |
| Memory | 32 GB (I) | 8 GB. One 8 GB DDR4 module in A1; 3 of 4 slots empty | E | **Wrong** |
| RAID controller | "Dell PERC", model not stated (I) | PERC H330 Adapter, PCIe slot 2, firmware 25.5.9.0001, no cache | E | Recorded |
| Virtual disks | RAID1 (I) | None | E | **Wrong** |
| Drive bays | — | 8× 3.5-inch. Two carriers fitted, one with a 2.5-inch adapter | E, R | Recorded |
| Disks | 600 GB SAS 10K installed; A400 removed (O); 2× HDD (I) | One disk: Seagate ST600MM0006, 600 GB, SAS, HDD, bay 0, non-RAID. The A400 is not installed | E, R | Confirmed (O); **Wrong** (I) |
| iDRAC | iDRAC8 Enterprise (I) | iDRAC8 Express, firmware 2.86.86.86 | E | **Wrong** |
| iDRAC port | — | No dedicated port. Shares LOM1 | R | Finding 2 |
| iDRAC network | — | LAN enabled. Address 10.10.99.2. Name `idrac-dc01` | E | Finding 2 |
| BIOS | — | 2.20.0 (2024-02-22) | E | Recorded |
| Network interfaces | 2 (P) | 2× Broadcom BCM5720 | E | Confirmed |
| LOM2 cabled | No (P) | Not cabled | R | Confirmed |
| Power supplies | 495 W (I) | One 495 W supply in slot 1. Slot 2 empty | E | Confirmed; Finding 4 |
| Operating system | Windows Server 2025 (I); none (O) | None (baseline) | — | See Observation 2 |
| Rack position | U01–U11 (I) | U01–U11. Tower on the rack floor. Chassis height 5U | E, R | Confirmed |

#### itc-uvy-esxi01

| Field | Claimed | As found | Evidence | Result |
|---|---|---|---|---|
| Model | Dell PowerEdge R620 (I) | Dell PowerEdge R620 | E | Confirmed |
| Service tag | — | ***L5Y1 | E | Recorded |
| CPU | 2× Xeon E5-2660 v2 (I) | 2× Xeon E5-2660 (first generation), 8 cores and 16 threads each | E | **Wrong** |
| Memory | 192 GB (I) | 304 GB. All 24 slots: 14× 16 GB and 10× 8 GB, mixed makes and speeds | E | **Wrong** |
| RAID controller | PERC H710 Mini (I) | PERC H710 Mini, firmware 21.3.5-0002, 512 MB cache | E | Confirmed |
| Virtual disks | RAID0 (I) | One RAID0 virtual disk on one disk | E | Confirmed; Finding 1 |
| Drive bays | — | Backplane with 4 slots | E | Recorded |
| Disks | Samsung 870 EVO 1 TB (O) | Samsung 870 EVO, 2 TB, SATA SSD, bay 0 | E, R | **Wrong** (size) |
| iDRAC | iDRAC7 Enterprise (I) | iDRAC7 Enterprise, firmware 2.65.65.65 | E | Confirmed |
| iDRAC port | — | Dedicated | R | Recorded |
| iDRAC network | — | LAN enabled. No address. Default name `idrac-***L5Y1` | E | Recorded |
| BIOS | — | 2.9.0 (2019-12-06) | E | Recorded |
| Network interfaces | 8: 4 onboard + 4 PCIe (P) | 4× Intel I350 onboard; Intel Gigabit ET quad-port adapter in slot 3 | E | Confirmed |
| Power supplies | 2 (I) | 2× 750 W. Supply 2 not powered | E, R | Confirmed; Finding 4 |
| Operating system | ESXi 8.0 (I); none (O) | None (baseline) | — | See Observation 2 |
| Rack position | U14 (I) | U14 | R | Confirmed |

#### itc-uvy-esxi02

| Field | Claimed | As found | Evidence | Result |
|---|---|---|---|---|
| Model | Dell PowerEdge R620 (I) | Dell PowerEdge R620 | E | Confirmed |
| Service tag | — | ***18X1 | E | Recorded |
| CPU | 2× Xeon E5-2660 v2 (I) | 2× Xeon E5-2670 (first generation), 8 cores and 16 threads each | E | **Wrong** |
| Memory | 192 GB (I) | 96 GB. 12 of 24 slots: 12× 8 GB at 1333 MHz | E | **Wrong** |
| RAID controller | PERC H710 Mini (I) | PERC H310 Mini, firmware 20.13.3-0001, no cache | E | **Wrong** |
| Virtual disks | RAID0 (I) | One RAID0 virtual disk on one disk | E | Confirmed; Finding 1 |
| Drive bays | — | Backplane with 8 slots | E | Recorded |
| Disks | A400 480 GB (O); Samsung 870 (I) | Kingston A400, 480 GB, SATA SSD, slot 0 | E, R | Confirmed (O); **Wrong** (I) |
| iDRAC | iDRAC7 Enterprise (I) | iDRAC7 Enterprise, firmware 2.65.65.65 | E | Confirmed |
| iDRAC port | — | Dedicated | R | Recorded |
| iDRAC network | — | LAN enabled. Address 192.168.0.196. Name `itc-uvy-idrac-prx-01` | E | Finding 3 |
| BIOS | — | 2.9.0 (2019-12-06) | E | Recorded |
| Network interfaces | 4 onboard, no PCIe card (P) | 4× Broadcom BCM5720, no PCIe card | E | Confirmed |
| Power supplies | 2 (I) | 2× 750 W. Supply 2 not powered. Owner states both are functional | E, R | Confirmed; Finding 4 |
| Operating system | none (O) | None (baseline) | — | See Observation 2 |
| Rack position | U13 (I) | U13 | R | Confirmed |

#### itc-uvy-ms01

| Field | Claimed | As found | Evidence | Result |
|---|---|---|---|---|
| Model | HPE ProLiant DL360 Gen9 (I, P); DL380 (O) | HP ProLiant DL360 Gen9 | S | Confirmed (I, P); **Wrong** (O) |
| Serial number | — | ******02TB (pull tab) | L | Recorded |
| Product number | — | 755262-B21 (pull tab) | L | Recorded |
| Serial and product ID in BIOS | — | Both blank | S, R | Finding 5 |
| CPU | 2× Xeon E5-2630 v3 (I) | 2× Xeon E5-2630 v3 @ 2.40 GHz | S | Confirmed |
| Memory | 256 GB (I) | 256 GB | S | Confirmed |
| RAID controller | Smart Array P440ar (I) | Smart Array P440ar, embedded | S | Confirmed |
| Arrays | RAID0 (I) | 1 array, 1 logical drive, 1 drive. A single-drive logical drive is RAID0 | S | Confirmed; Finding 1 |
| Disks | Samsung 870 EVO 1 TB (I, O) | 1 TB SATA SSD, Port 1I : Box 1 : Bay 2. Model not shown on screen. No wear-out reported | S | Size confirmed; model not checked |
| iLO | iLO 4, 2.82 (I) | iLO 4, firmware 2.82, Advanced licence | S, R | Confirmed |
| iLO port | — | Dedicated | R | Recorded |
| iLO network | — | No IPv4 address | S | Recorded |
| BIOS | P89, 2024-08-29 (I) | P89 v3.40 (2024-08-29). Backup P89 v3.30 (2023-09-21) | S | Confirmed |
| Boot mode | — | UEFI | S | Recorded |
| Network interfaces | 4 (P) | 4 embedded LOM ports | S, R | Confirmed |
| Power supplies | — | 2× HPE 500 W Platinum, P/N 723595-101, option kit 720478-B21. Both powered | L, R | Recorded |
| Operating system | Windows Server 2025 Datacenter (I); none (O) | None (baseline) | — | — |
| Rack position | U12 (I) | U12 | R | Confirmed |

#### Not checked — session ended by the owner after the servers

| Item | Reason |
|---|---|
| itc-uvy-core01, itc-uvy-oob01, itc-uvy-rtr01: model, serial, rack position; core01 uplink bay and power supplies | Session ended |
| Patch panel, PDU-01, PDU-02, UPS | Session ended |
| Spare disks (A400 480 GB ex-dc01; 3× 300 GB SAS 10K, owner statement) | Session ended. Loose disks are not visible to any controller |
| USB-serial console cables | Session ended |
| PAW-01 (Lenovo T470s), test host (ASUS PN52) | Session ended |

### 6.3 Findings requiring a change

None of these is fixed in this record. Each is tracked as a card on the project board.

**Finding 1 — no server has disk redundancy.**
dc01 has one non-RAID disk and no virtual disk. esxi01, esxi02 and ms01 each run one RAID0 volume on one disk. One disk failure loses the host's storage.
Evidence: the controller reports in 6.2.
The owner accepts this for now under the budget constraint in the Project Charter. The fix is tracked on the project board.

**Finding 2 — dc01 has no out-of-band path, and its BMC sits on a production interface.**
The iDRAC is Express with no dedicated port. It shares LOM1, which the Physical Port Map patches to itc-uvy-core01 Gi1/0/10. Its LAN access is enabled with leftover configuration: address 10.10.99.2 and name `idrac-dc01`. The address is outside the management subnet, 10.10.0.0/24. The name does not follow the project's `itc-uvy-` pattern. The Segmentation Design places every BMC in the management segment.
Exposure today: Device Configuration 1.2 holds Gi1/0/10 shut, so the iDRAC cannot be reached over the network. Port state was not checked in this session.
Evidence: the export (Express, LAN, address, name) and the owner's reading of the NIC selection.

**Finding 3 — esxi02's iDRAC carries leftover configuration from a previous lab.**
LAN access is enabled with address 192.168.0.196, a typical home-network range outside the lab's addressing, and name `itc-uvy-idrac-prx-01`. The Physical Port Map connects this iDRAC to itc-uvy-oob01 Gi0/2, in the management segment.
Exposure today: Device Configuration 1.2 holds Gi0/2 shut. Port state was not checked in this session.
Evidence: the export.

**Finding 4 — power supply redundancy is not in place.**
esxi01 and esxi02 each have two supplies, but only supply 1 was powered. dc01 has one supply and an empty second slot. ms01 is the only server with both supplies powered.
Evidence: the exports and owner readings.
Update 2026-10-08: the owner states that both supplies are now connected on every dual-supply device, done during rack cable work on 2026-10-06/07. Not verified. To be checked in the next survey.

**Finding 5 — ms01's system board does not carry the chassis identity.**
The serial number and product ID are blank in the BIOS. The pull tab shows the serial (ending 02TB) and product number 755262-B21.
Evidence: screen and label photos.

**Finding 6 — the previous inventory sheet is unreliable.**
It was wrong on at least one of CPU, memory, RAID controller, disks or BMC licence for three of the four servers. The two R620s were recorded as identical. They differ in CPU, memory and RAID controller.
Evidence: section 6.2.

### 6.4 Deviations

**Deviation 1 — the session started before its checklist was agreed, and earlier than planned.** The checklist was a draft. The session started before the planned late-evening start.

**Deviation 2 — hardware was handled during a read-only session.** One ms01 power supply was removed while the server was powered, to photograph its label. It was reinstalled. Both supplies were powered at the end. The same data was available from the BIOS and iLO screens without touching the hardware.

**Deviation 3 — hardware was changed before the session and not recorded.** The 600 GB SAS disk was installed in dc01, and the A400 was removed from dc01. No change process for hardware existed at the time, and the change was not recorded anywhere.

**Deviation 4 — the USB stick was not identified.** The record does not state which stick carried the exports.

### 6.5 Observations

1. **BMC clocks are not trustworthy.** The dc01 export file name gives 10:51. The timestamps inside it give 15:22 and 16:22. BMC times were not used as session times.
2. **Earlier installations left traces.** The last-known host names were `WIN-KKQILQP5187` on dc01, `itc-uvy-esxi01.lab.theitchef.com` on esxi01, and `MINWINPC` on esxi02 (the default name of Windows PE, the installer environment). The baseline of no operating system stands. The existing RAID0 volumes on esxi01, esxi02 and ms01 still exist.
3. **esxi01 mixes memory modules.** Sizes, makes and rated speeds differ. All modules run at the slowest common speed.
4. **esxi02's controller has no cache.** The H310 writes straight to disk. It can also present disks as non-RAID.
5. **Both R620s have first-generation E5 processors,** not v2. Any compatibility check that assumed v2 must be redone.
6. **dc01 cannot take more 2.5-inch disks without parts.** Each needs a carrier and a 2.5-inch adapter. None are in stock.

### 6.6 State at end of session

- No configuration was changed on any server (owner statement).
- ms01: both power supplies installed and powered.
- esxi01 and esxi02: supply 1 powered, supply 2 not powered.
- dc01: one supply.
- Server power state at the end of the session was not recorded.
- Network devices, passive equipment, spares and endpoints were not touched.

## 7. Acceptance criteria

- Every planned item has an as-found value or a stated reason for not being checked.
- Every as-found value names its evidence source.
- Every prior claim is marked confirmed or wrong, with its source.
- Findings are recorded with their evidence. None are fixed in this record.
- Every departure from the plan or the change process is recorded as a deviation.
- The end state is stated, including what was not recorded.
- No MAC address and no full serial number appears in the record.
- The record was written on the day of the session. Any later addition is dated.

## 8. References

- **Physical Port Map** — the interface and cabling claims checked.
- **Segmentation Design** — the placement of BMCs in the management segment.
- **VLAN and IP Address Plan** — the management subnet.
- **Device Configuration and Bring-Up** — the switch ports held shut (Findings 2 and 3).
- **Project Charter** — the budget constraint behind the accepted risk in Finding 1.
- **Change-Management SOP** — its scope excludes hardware changes, the gap behind Deviation 3.
- **Master Document** — where this document sits in the repository.