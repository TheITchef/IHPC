# Operational Change Procedure — defines how changes to the lab's hardware and configuration are planned, approved, made and recorded.

**Status:** Draft · **Version:** 1.1 · **Last updated:** 2026-10-13 · **Owner:** Ioannis Mintzivyris

## 2. Overview

This procedure governs every change to the lab itself: its devices, cabling, power and configuration. A change is planned before it is made, approved at the level its risk requires, made with a way back where one exists — and with the point of no return named where it does not — and checked afterwards. Every change leaves a change record.

The procedure follows ITIL 4 change enablement (the industry's standard practice for IT change), scaled to a single operator. Where a company relies on a second person to approve a change, this procedure relies on a written plan reviewed before any work starts.

## 3. Scope

### In scope

- Any modification to a lab device or its connections: hardware, cabling, power, firmware, BIOS and BMC settings, and device configuration.
- Physical work inside the rack, even when no connection is meant to change.

### Out of scope

- Changes to project documents (see Change-Management SOP).
- Privileged sessions that change nothing (see Privileged Access Log).
- Read-only surveys and inspections. These are recorded as records, not changes.
- Changes to PAW-01 or the PN52 themselves. They are workstations, not lab infrastructure.

## 4. Dependencies

- **Documentation Standard** (Approved 1.0) — change records are records (6.1), and Rules 6 and 7 apply to them.
- **Change-Management SOP** (Approved 1.0) — the review process a normal change record passes through before work starts.
- **Privileged Access Log** (Approved 1.0) — every change made over a privileged session is linked to its log entry.
- **Physical Port Map** (Approved 1.2) — the reference that cable and port work is verified against.

## 5. Deliverables

- Three change types, each with its approval path.
- A change record template.
- A standard change catalogue, with its first entries.
- Rules for emergency changes and break-glass use.
- Rules for changes discovered after they were made.
- The location and numbering of change records.

## 6. Detailed content

### 6.1 What counts as a change

A change is any modification to a lab device or its connections:

- **Hardware:** adding, removing or moving a component, such as a disk, power supply or card.
- **Cabling:** connecting, disconnecting, re-routing or re-seating any cable, including power cords.
- **Firmware:** any update or downgrade.
- **Settings:** BIOS settings (the server's start-up firmware), BMC settings (the management chip built into each server), and RAID controller settings (the disk controller).
- **Configuration:** any change to a device's running configuration (the settings in use) or saved configuration (the settings loaded at start-up).

Powering a device on or off is not a change. Reading a device is not a change.

When in doubt, it is a change.

### 6.2 Change types

Every change is one of three types (ITIL 4 terms). The type decides how it is approved and how much it records.

| Type | When it applies | Approval | Record |
|---|---|---|---|
| Standard | The work is listed in the catalogue (6.6). It is low-risk, proven, and done the same way every time. | Pre-approved by its catalogue entry. | Short: the catalogue entry used, the date, and the verification result. |
| Normal | Any change that is not standard and not an emergency. | The change record is reviewed and approved before any work starts. | Full change record (6.4). |
| Emergency | Waiting for review would cause harm: a security exposure, or loss of control of a device. | The owner decides at the time. | Full change record within 24 hours, including a post-implementation review. |

Inconvenience is not an emergency. A change that could wait for review is a normal change.

### 6.3 Priority

Every change is given a priority from two separate judgements, recorded in its change record. Problems found that are not yet changes are tracked as cards on the project board.

The two judgements are:

- **Impact** — how much harm, to what. High: exposure of Tier 0 (the management segment that controls every device) or of the internet edge, or loss of control of a device. Medium: a segment or service down. Low: cosmetic or inconvenience.
- **Urgency** — how fast the harm grows while waiting. High: harm is happening now and growing. Medium: harm within days. Low: no harm while waiting.

| | Urgency high | Urgency medium | Urgency low |
|---|---|---|---|
| Impact high | P1 | P2 | P3 |
| Impact medium | P2 | P3 | P4 |
| Impact low | P3 | P4 | P4 |

Only P1 qualifies as an emergency change.

**Containment.** A free and reversible action that stops harm — unplugging a cable, powering a device off, shutting a port — may be taken at once, at any priority. It is recorded in the change record of the change that follows.

### 6.4 The change record

A change record follows the Documentation Standard's eight parts. Its Detailed content always contains these subsections, in this order:

| Subsection | What it holds |
|---|---|
| Summary | Change ID, type, priority, devices affected, date raised, planned date, and the Privileged Access Log entry if a session is used. |
| Reason | Why the change is needed, with a pointer to the card, finding or design that requires it. |
| Risk and impact | What could break, how likely it is (low, medium, high), and what stops working if it does. |
| Before state | How the current state is captured before work starts: a saved configuration, photos, or the relevant Port Map rows. |
| Steps | The work, in order, each step with its own check. These steps are the method of procedure (MOP) for this change. |
| Backout plan | The condition that triggers a backout, and the steps that return the device to its before state. If any step cannot be undone, the plan names it as the point of no return and states how the device is recovered after it. |
| Verification | The checks that prove the change worked, each with its expected result. |
| Outcome | Written after the work: actual start and end, what was done, any departure from the steps, and the verification results. Dated. |
| Post-implementation review | Written after the work: did it achieve its reason, and what changes next time. Dated. |

A change record is written in two stages. The plan (Summary to Verification) is reviewed and approved before work starts. The Outcome and the review are added afterwards, as dated additions.

### 6.5 How a normal change moves

1. **Card.** The change has a card on the project board before anything else.
2. **Plan.** The change record is written as Draft on its own branch, with Summary to Verification complete.
3. **Plan review.** The record is opened for review. When the plan passes, the approval is recorded on the review with its date, and the record's status becomes Approved. No work starts before this point.
4. **Work.** The change is made following the Steps. Where it needs access to a device's settings or configuration, it is made in a logged privileged session. If any step fails its check, or anything unplanned appears, work stops and one of the responses below is chosen.
5. **Outcome.** The Outcome and the Post-implementation review are added to the same record, dated. The Outcome states one of three results: completed, backed out, or failed.
6. **Close.** The record is merged and the card moves to Done. A backed-out or failed change is still closed this way; its record is the evidence of what happened.

**When work stops.** The owner chooses one of three responses and records the choice in the Outcome.

| Response | When | What follows |
|---|---|---|
| Back out | The change cannot be completed as planned. | The backout steps are run and the before state is verified. Result: backed out. |
| Hold | It is safe to pause, and the cause is not clear. | The device is left in a known, safe state and the session ends. The change stays open. |
| Continue | The deviation does not affect what the failed check protects. | The deviation is recorded in the Outcome, and work continues. |

A different fix made on the spot is not a fourth response. It is a new change, with its own record, or an emergency change if it is P1.

### 6.6 Standard change catalogue

A standard change is pre-approved by its entry here. It is recorded in the Privileged Access Log entry of the session it is made in: the catalogue ID and the verification result.

A kind of change joins this catalogue only after it has been made successfully at least once as a normal change. Until then it is listed as a candidate. Adding or activating an entry is a revision of this procedure.

**SC-01 — Open the management switch port for a planned session**

| Field | Content |
|---|---|
| Purpose | Connect PAW-01 (the privileged workstation) to the management segment when a task needs it. |
| Device | itc-uvy-oob01, port Gi0/8. |
| Steps | Enable the port at the start of the session. Shut it at the end. Do not save the running configuration, so a restart returns the port to shut. |
| Verification | At start: the port is up and connected. At end: the port is administratively down. |
| Backout | Shut the port. |
| Risk | Low. The port connects one known workstation to its own segment, for one logged session. |
| Status | Candidate. Becomes active after its first successful use as a normal change. |

### 6.7 Emergency changes and break-glass

An emergency change is allowed only at priority P1 (6.3).

- Contain first, where a free and reversible action exists.
- Make the smallest change that stops the harm. Anything more waits for a normal change.
- Write the full change record within 24 hours, including the Post-implementation review. The review states why the change could not wait.

**Break-glass workstation.** If PAW-01 cannot be used, the ASUS PN52 may be used for an emergency change. It is a general-purpose machine, not a hardened workstation; this is an accepted risk. While it is connected to the lab, its Wi-Fi is off and nothing else is connected to it. The session is logged in the Privileged Access Log as a break-glass session, naming the PN52.

### 6.8 Changes found after they were made

A change made without a change record is recorded as soon as it is found.

- The record is a normal change record, marked "Recorded after the fact" in its Summary.
- It states what changed, when, and by whom, as far as known, and says plainly what is not known.
- Its Verification checks the current state of what was changed.
- Its Post-implementation review states why the change went unrecorded.
- Its raised date is the day the record is written. The date of the change itself goes in the Outcome, as far as known — never backdated into the raised date.

### 6.9 Where change records are kept

Change records are kept in `docs/changes/`. Each is named `CR-NNNN-short-title.md`, numbered in the order the records are created. A number is never reused, even if the change is cancelled.

## 7. Acceptance criteria

- Every kind of change listed in 6.1 falls under one of the types in 6.2.
- An emergency is defined by priority (6.3), not by judgement at the time.
- The change record template lists every subsection and what it holds, including its dates.
- The normal change process states that no work starts before the plan is approved.
- Every catalogue entry states its purpose, device, steps, verification, backout, risk and status.
- Emergency rules state containment, the time limit for the record, and the conditions for the break-glass workstation.
- Changes found afterwards are recorded without backdating.
- The location and numbering of change records are stated.
- The responses to a stopped change are stated, and an unplanned fix on the spot is excluded.
- A step that cannot be undone is named in the plan as the point of no return.
- The procedure names no specific tools.

## 8. References

- **Documentation Standard** — the record type and the writing rules change records follow.
- **Change-Management SOP** — the review process a change record passes through.
- **Privileged Access Log** — where sessions and standard changes are recorded.
- **Physical Port Map** — the reference for cable and port work.
- **Device Configuration and Bring-Up** — holds itc-uvy-oob01 Gi0/8 shut (SC-01).
- **Project Charter** — the single-operator constraint behind the approval model.
- **Master Document** — where this procedure sits in the repository.