# CR-0002 Rack Power Cabling — records, after the fact, the re-routing of all power cables in the rack and the connection of every second power supply.

**Status:** Draft · **Version:** 0.1 · **Last updated:** 2026-10-21 · **Owner:** Ioannis Mintzivyris

## 2. Overview

On 2026-10-06 and 2026-10-07 the owner reworked the power cabling at the back of the rack. Every power cable was unplugged, re-routed down the left side to the PDUs (power distribution units, the rack's power strips), and fixed with hooks and Velcro straps. The second power supply of every dual-supply device was connected. No change record was written at the time.

This record is written after the fact, as the Operational Change Procedure requires. The work has not been verified. Verification is set out here and carried out at the next rack survey.

## 3. Scope

### In scope

- The power cabling work in the rack on 2026-10-06 and 2026-10-07.

### Out of scope

- The power design: which supply is fed from which PDU, and from which power line. Tracked by the PSU Cabling and Rack Elevation and Power Diagram cards on the project board.
- Data cabling rules. Tracked by the Rack Cabling Standard card.

## 4. Dependencies

- **Operational Change Procedure** (Approved 1.1) — section 6.8, changes found after they were made.
- **Rack Inventory Verification Run** (Approved 1.0) — the server power supply state before this change, and Finding 4.
- **Physical Port Map** (Approved 1.2) — the reference the data links are checked against.

## 5. Deliverables

- One change record, marked "Recorded after the fact", with the verification still to be done and a post-implementation review.

## 6. Detailed content

### 6.1 Summary

| Field | Content |
|---|---|
| Change ID | CR-0002 |
| Recorded after the fact | Yes |
| Type | Normal |
| Priority | Not assessed. Priorities did not exist when the change was made. |
| Devices affected | Every powered device in the rack (power cabling only) |
| Date raised | 2026-10-21 |
| Planned date | None. The change was not planned. |
| Privileged Access Log entry | None. The log did not exist at the time. |

### 6.2 Reason

The power cables at the back of the rack were tangled. The work was done to bring them into order (owner).

### 6.3 Risk and impact

Assessed after the fact.

- **During the work:** every power cable was unplugged and plugged back in. A wrong or loose connection could leave a device or a power supply without power. The only check was the owner powering the devices up afterwards.
- **After the work:** every power supply is fed from the same PDU, and the rack has a single power line (owner). A second power supply now protects a device against the failure of one supply. It does not protect against the failure of the PDU or the power line: either one stops every device in the rack.
- **Power path:** each PDU feeds two 8-port extension strips, and the devices plug into the strips (owner). The ratings of the strips against the load are not known.

### 6.4 Before state

Not captured.

Known from the Rack Inventory Verification Run (6.2, 2026-10-02): on esxi01 and esxi02 only supply 1 was powered; ms01 had both supplies powered; dc01 has one supply. The network devices were not checked.

Not known: which PDU fed each supply before the change, and how the cables were routed.

### 6.5 Steps

None written. The change was not planned.

As far as known:

1. Every power cable was unplugged.
2. The cables were re-routed down the left side of the rack, along the PDUs.
3. Hooks and Velcro straps were installed to hold them.
4. Every supply, including every second supply, was plugged into the extension strips of the upper PDU.
5. The devices were powered up to confirm they ran.

### 6.6 Backout plan

None written.

The change is reversible. There is no point of no return.

### 6.7 Verification

To be carried out at the Second Rack Survey. Results are added to this record as a dated addition.

| Check | Expected | Result |
|---|---|---|
| Every supply of every dual-supply device is powered | Both supplies reported powered, on the device's management screen or its supply indicator | Not verified. The owner saw all devices and supplies on at power-up. |
| Each supply is traced to its PDU | Every supply on the upper PDU, as stated | Not verified. |
| No data link was changed | Every data link matches the Physical Port Map | Not verified. The owner states no data cable endpoint changed (2026-10-08). |

### 6.8 Outcome

Written 2026-10-21.

- **Date of change:** 2026-10-06 and 2026-10-07 (owner). Start and end times not known.
- **Done by:** the owner.
- **What was done:** see 6.5.
- **Departures from the steps:** none can be stated. There were no written steps.
- **Result:** completed (owner). Not yet verified; see 6.7.
- **Not known:** start and end times; the PDU each supply used before the change; which network devices have two supplies.

### 6.9 Post-implementation review

Written 2026-10-21.

- **Did it achieve its reason?** The owner states the cables are now in order. This is not yet verified.
- **Effect on Finding 4 of the Rack Inventory Verification Run:** the second supplies are connected, so a single supply failure no longer stops a dual-supply server. Power redundancy is still incomplete: every supply shares one PDU and one power line. Finding 4 stays open until 6.7 is verified.
- **Why it went unrecorded:** no procedure covered hardware changes at the time. The Operational Change Procedure came into force on 2026-10-13. This record was written eight days later.
- **What changes next time:** power cabling work is a normal change. It needs a card, an approved plan with a before state (photos and a list of which supply is on which PDU) and a backout plan, before any cable is unplugged.

## 7. Acceptance criteria

- The Summary marks the record "Recorded after the fact".
- What changed, when, and by whom are stated as far as known, and what is not known is listed.
- The raised date is the day the record was written. The date of the change is stated in the Outcome and is never used as the raised date.
- Every verification check states what is expected, and states plainly that it has not been done.
- The risk left after the change is stated.
- The post-implementation review states why the change went unrecorded.

## 8. References

- **Operational Change Procedure** — section 6.8, the rules this record follows.
- **CR-0003 Rack Data Cabling** — the data cabling work that followed this one.
- **Rack Inventory Verification Run** — the supply state before the change, and Finding 4.
- **Physical Port Map** — the reference for the data link check.
- **Master Document** — where this record sits in the repository.
