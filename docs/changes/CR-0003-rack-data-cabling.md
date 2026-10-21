# CR-0003 Rack Data Cabling — records, after the fact, the re-routing of every data cable in the rack.

**Status:** Draft · **Version:** 0.1 · **Last updated:** 2026-10-21 · **Owner:** Ioannis Mintzivyris

## 2. Overview

On or about 2026-10-09 the owner re-routed every data cable at the back of the rack. Each cable was unplugged in turn, routed down the right side of the rack, away from the power cables, and plugged back in. Cable labels were updated. No change record was written at the time.

This work followed the power cabling rework recorded in CR-0002, as a separate session. This record is written after the fact, as the Operational Change Procedure requires. The work has not been verified. Verification is set out here and carried out at the next rack survey.

## 3. Scope

### In scope

- The data cabling work in the rack on or about 2026-10-09, including the label updates.

### Out of scope

- A second patch panel, further separation of server and network cabling, and coloured patch cables. These are ideas for the future, tracked by the Rack Cabling Standard card on the project board.
- The labelling scheme. Tracked by the Cable Labelling card.
- The power cabling. Recorded in CR-0002.

## 4. Dependencies

- **Operational Change Procedure** (Approved 1.1) — section 6.8, changes found after they were made.
- **Physical Port Map** (Approved 1.2) — the intended data links, which the verification checks against.
- **Privileged Access Log** (Approved 1.0) — the disconnected baseline of 2026-10-08, which this work may have touched.

## 5. Deliverables

- One change record, marked "Recorded after the fact", with the verification still to be done and a post-implementation review.

## 6. Detailed content

### 6.1 Summary

| Field | Content |
|---|---|
| Change ID | CR-0003 |
| Recorded after the fact | Yes |
| Type | Normal |
| Priority | Not assessed. Priorities did not exist when the change was made. |
| Devices affected | Every device with a data cable, and the patch panel |
| Date raised | 2026-10-21 |
| Planned date | None. The change was not planned. |
| Privileged Access Log entry | None. The work was physical; no session is recorded. |

### 6.2 Reason

The cables at the back of the rack were tangled. Data cables were moved to the opposite side from the power cables (owner). This completes the clean-up begun in CR-0002.

### 6.3 Risk and impact

Assessed after the fact.

- **During the work:** every data cable was unplugged and plugged back in. A cable returned to the wrong port changes what that port connects to. In a segmented network, that can join two segments that are meant to be apart. Working one cable at a time reduced this risk. It did not remove it.
- **After the work:** whether every link matches the Physical Port Map is not known. A green link light shows that a cable is connected at both ends. It does not show that it is connected to the right port.
- **Labels:** labels were changed, and the labelling scheme is not written down. A label cannot be checked against a rule that does not exist.

### 6.4 Before state

Not captured.

Known: the Physical Port Map defines the intended links. The Privileged Access Log baseline of 2026-10-08 records the itc-uvy-oob01 Gi0/8 cable present at the switch with its far end unplugged, and console cable 1 attached to the itc-uvy-core01 console port.

Not known: whether the links matched the Physical Port Map before the change. No link trace has been done. The survey of 2026-10-02 did not cover cabling.

### 6.5 Steps

None written. The change was not planned.

As far as known:

1. Each data cable was unplugged, one at a time.
2. It was routed down the right side of the rack, opposite the power cables, and held with hooks and Velcro straps.
3. It was plugged back into its port.
4. Cable labels were updated.
5. The link lights on the connected devices were checked. All were green.

### 6.6 Backout plan

None written.

The change is reversible. There is no point of no return.

### 6.7 Verification

To be carried out at the Second Rack Survey. Results are added to this record as a dated addition.

| Check | Expected | Result |
|---|---|---|
| Every data link is traced end to end | Every link matches the Physical Port Map | Not verified. |
| Every label matches the cable it is on | Label and traced link agree | Not verified. |
| The Privileged Access Log baseline still holds | Gi0/8 cable far end unplugged; console cable 1 attached at the itc-uvy-core01 console port, workstation end unplugged | Not verified. |
| Link lights | Lit on every connected port | Owner: all green after the work. This shows a connection, not the right connection. |

### 6.8 Outcome

Written 2026-10-21.

- **Date of change:** 2026-10-09, as far as the owner recalls. Not certain. Start and end times not known.
- **Done by:** the owner.
- **What was done:** see 6.5.
- **Departures from the steps:** none can be stated. There were no written steps.
- **Result:** completed (owner). Not yet verified; see 6.7.
- **Not known:** the exact date; start and end times; whether the rack was powered while cables were unplugged; whether any cable went back to a different port; what the labels said before and after; whether the console cables or the Gi0/8 cable were touched.

### 6.9 Post-implementation review

Written 2026-10-21.

- **Did it achieve its reason?** The owner states the data cables are now separated from the power cables. This is not yet verified.
- **Why it went unrecorded:** no procedure covered hardware changes at the time. The Operational Change Procedure came into force on 2026-10-13. This record was written eight days later.
- **What changes next time:** data cabling work is a normal change. Its before state is a link trace or a photo of every port. Afterwards, every moved link is traced against the Physical Port Map. Label changes wait for a written labelling scheme.

## 7. Acceptance criteria

- The Summary marks the record "Recorded after the fact".
- What changed, when, and by whom are stated as far as known, and what is not known is listed, including the uncertain date.
- The raised date is the day the record was written. The date of the change appears only in the Outcome.
- Every verification check states what is expected, and states plainly whether it has been done.
- The risk left after the change is stated.
- The post-implementation review states why the change went unrecorded.

## 8. References

- **Operational Change Procedure** — section 6.8, the rules this record follows.
- **CR-0002 Rack Power Cabling** — the power cabling work that came before this one.
- **Physical Port Map** — the intended data links.
- **Privileged Access Log** — the disconnected baseline of 2026-10-08.
- **Master Document** — where this record sits in the repository.
