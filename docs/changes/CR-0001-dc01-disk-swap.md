# CR-0001 dc01 Disk Swap — records, after the fact, the replacement of dc01's SSD with a SAS hard disk.

**Status:** Draft · **Version:** 0.1 · **Last updated:** 2026-10-21 · **Owner:** Ioannis Mintzivyris

## 2. Overview

On 2026-09-30 the owner removed the Kingston A400 SSD (solid-state disk) from itc-uvy-dc01 and fitted a 600 GB SAS hard disk (SAS: the server-grade disk interface) in its place. No change record was written at the time, because no procedure for hardware changes existed.

This record is written after the fact, as the Operational Change Procedure requires. The change was verified by the rack survey of 2026-10-02.

## 3. Scope

### In scope

- The disk change in itc-uvy-dc01 on 2026-09-30.

### Out of scope

- Where the freed A400 is used next, and disk redundancy for dc01. Both are decided by the Storage Redundancy ADR card on the project board.
- Any operating system or role on dc01.

## 4. Dependencies

- **Operational Change Procedure** (Approved 1.1) — section 6.8, changes found after they were made.
- **Rack Inventory Verification Run** (Approved 1.0) — the evidence used for Verification.

## 5. Deliverables

- One change record, marked "Recorded after the fact", with its verification evidence and a post-implementation review.

## 6. Detailed content

### 6.1 Summary

| Field | Content |
|---|---|
| Change ID | CR-0001 |
| Recorded after the fact | Yes |
| Type | Normal |
| Priority | Not assessed. Priorities did not exist when the change was made. |
| Devices affected | itc-uvy-dc01 |
| Date raised | 2026-10-21 |
| Planned date | None. The change was not planned. |
| Privileged Access Log entry | None. The log did not exist at the time. |

### 6.2 Reason

To free the Kingston A400 480 GB SSD, so it can be paired with the identical A400 already in the lab as a RAID1 set (two disks holding identical copies), wherever that set is needed.

dc01 will run only as a domain controller. A SAS hard disk is enough for that role (owner).

### 6.3 Risk and impact

Assessed after the fact.

- No service ran on dc01, so no service could break.
- What the A400 held before removal is not known. The project treats all servers as blank (Rack Inventory Verification Run, 6.1).
- dc01 now runs on one disk with no copy. This is Finding 1 of the Rack Inventory Verification Run, accepted for now under the budget constraint.

### 6.4 Before state

Not captured.

Known: the A400 was installed in dc01 (owner).

Not known: the bay it was in, and how the RAID controller presented it.

The previous inventory sheet (2026-07-03) is not used as the before state. It was found unreliable (Rack Inventory Verification Run, Finding 6).

### 6.5 Steps

None written. The change was not planned.

As far as known: the A400 was removed, and the 600 GB SAS disk was fitted in bay 0.

### 6.6 Backout plan

None written.

The change is reversible: the A400 is kept as a spare. There is no point of no return.

### 6.7 Verification

| Check | Expected | Result |
|---|---|---|
| dc01's RAID controller reports the SAS disk | One 600 GB SAS disk in bay 0 | Confirmed, 2026-10-02 (Rack Inventory Verification Run, 6.2, dc01 Disks row) |
| The A400 is no longer in dc01 | Not reported by the controller | Confirmed, 2026-10-02 (same row) |

No later change to dc01's disks is known.

### 6.8 Outcome

Written 2026-10-21.

- **Date of change:** 2026-09-30 (owner). Start and end times not known.
- **Done by:** the owner.
- **What was done:** the A400 was removed from dc01 and the 600 GB SAS disk was fitted in bay 0.
- **Departures from the steps:** none can be stated. There were no written steps.
- **Result:** completed.
- **Not known:** start and end times; the bay the A400 was in; whether RAID controller settings were changed; what the A400 held.

### 6.9 Post-implementation review

Written 2026-10-21.

- **Did it achieve its reason?** In part. The A400 is free and dc01 runs on the SAS disk. The A400 has not yet been paired; that waits for the Storage Redundancy ADR.
- **Why it went unrecorded:** no procedure covered hardware changes. The Change-Management SOP covered project documents only (Rack Inventory Verification Run, Deviation 3). The Operational Change Procedure came into force on 2026-10-13. This record was written eight days later.
- **What changes next time:** hardware changes now follow the Operational Change Procedure. A disk change is a normal change: a card, an approved plan with a before state and backout, then the work.

## 7. Acceptance criteria

- The Summary marks the record "Recorded after the fact".
- What changed, when, and by whom are stated as far as known, and what is not known is listed.
- The raised date is the day the record was written. The date of the change is stated in the Outcome and is never used as the raised date.
- Verification checks the state after the change, with dated evidence.
- The post-implementation review states why the change went unrecorded.
- No full serial number appears in the record.

## 8. References

- **Operational Change Procedure** — section 6.8, the rules this record follows.
- **Rack Inventory Verification Run** — the verification evidence, Finding 1 and Deviation 3.
- **Project Charter** — the budget constraint behind the accepted risk.
- **Master Document** — where this record sits in the repository.
