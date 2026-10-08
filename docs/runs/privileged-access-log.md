# Privileged Access Log — records every privileged session into the lab, planned or emergency.

**Status:** Draft · **Version:** 0.1 · **Last updated:** 2026-10-08 · **Owner:** Ioannis Mintzivyris

## 2. Overview

Privileged access to the lab is disconnected by default. It is connected only for a specific task, then disconnected again. This is just-in-time access: no standing path exists between sessions.

PAW-01 (the Lenovo T470s) is the only workstation allowed this access. By default it reaches the network devices by console cable. It connects to the management switch only when a task needs the management network.

Every session is logged here as one of two types. A planned session is normal admin work. A break-glass session is an emergency, used when the normal path has failed. A session that is not in this log did not follow the rules.

## 3. Scope

### In scope

- Every privileged connection into the lab: console cables to lab devices, and the management switch port (itc-uvy-oob01 Gi0/8).
- The disconnected baseline these sessions start from and return to.

### Out of scope

- Test-host connections during bench sessions. Those are recorded in each session's run record.
- Hardening of PAW-01. That is Phase 3 work.

## 4. Dependencies

- **Validation and Handover** (Approved 1.2), section 6.9.2 — the decision that PAW-01 ends Phase 1 disconnected, with each connection deliberate and logged.
- **Routing and ACL Design** (Approved 1.1) — PAW-01 as the only trusted origin for management access.
- **Physical Port Map** (Approved 1.2) — the management switch port Gi0/8. The map still labels the T470s as PAW-02; that naming is corrected separately.

## 5. Deliverables

- **A baseline** — the verified disconnected state, dated.
- **An entry format** — the fields every session records.
- **The log** — one entry per session, newest last.

## 6. Detailed content

### 6.1 What counts as a session

- A session starts when any cable connects PAW-01 to a lab device or to itc-uvy-oob01 Gi0/8.
- It ends when every such cable is unplugged at the PAW-01 end and the baseline in 6.2 is restored.
- This holds whether the rack is powered or not.
- During a session, PAW-01 has no other network connection: Wi-Fi off.
- A break-glass session also records why the normal path failed, and a short review afterwards.

### 6.2 Baseline

Verified 2026-10-08, evening, by the owner at the rack and at the workstation.

| Item | State |
|---|---|
| PAW-01 network | No Ethernet connected. Internet through home Wi-Fi. |
| USB console hub | Unplugged from PAW-01. |
| Console cable 1 (USB mini-B) | Attached to the itc-uvy-core01 console port. Workstation end unplugged. |
| Console cable 2 | Not attached to any device. |
| itc-uvy-oob01 Gi0/8 | Cable present at the switch. Far end unplugged. |
| Rack | Powered off. |

Before this baseline, the USB console hub was plugged into PAW-01, with cable 1 attached to itc-uvy-core01. It was unplugged on 2026-10-08 to reach this state.

Not checked: the switch port state of Gi0/8. Device Configuration 1.2 holds it shut.

### 6.3 Entry format

One row per session. Rows are added at the bottom and never edited afterwards. A correction is a new row that names the row it corrects.

| Field | Content |
|---|---|
| # | Running number |
| Date | YYYY-MM-DD |
| Start – End | Local time |
| Type | Planned or Break-glass |
| Path | Console to which device(s), and/or Gi0/8 |
| Task | One line, with a pointer to its card or run record |
| Wi-Fi off | Yes / No |
| Changes made | None, or a pointer to the change record |
| Baseline restored | Yes, with time |
| Review | Break-glass only: why the normal path failed, and what follows |

No password, key or address of a lab device is written in this log.

### 6.4 Log

Sessions before 2026-10-08 were not logged. The known ones are recorded in the run records of 2026-09-11 and 2026-09-18. The device check of 2026-09-28 has no record.

| # | Date | Start – End | Type | Path | Task | Wi-Fi off | Changes made | Baseline restored | Review |
|---|---|---|---|---|---|---|---|---|---|

No sessions logged yet.

## 7. Acceptance criteria

- The baseline is verified, dated, and states what was not checked.
- Every field in the entry format can be filled with a fact, not an opinion.
- The rules for what starts and ends a session are stated.
- The log is append-only: corrections are new rows, never edits.
- Break-glass entries carry a reason and a review.
- No password, key or device address appears in the log.

## 8. References

- **Validation and Handover** — section 6.9.2, the end-state decision for PAW-01.
- **Routing and ACL Design** — PAW-01 as the only trusted origin for management access.
- **Physical Port Map** — the management switch port Gi0/8.
- **Device Configuration and Bring-Up** — holds Gi0/8 shut.
- **Master Document** — where this document sits in the repository.