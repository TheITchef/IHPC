---

kanban-plugin: board

---

## Backlog

- [ ] [[Replacement VLAN 20 test host (blocks validation)]]
    - Why: the PN52 was the VLAN 20 test host and is sold. Validation (E2) needs a host in VLAN 20.
    - Done when: a replacement is chosen and named on this card.
- [ ] [[Network device hardening - console auth, exec-timeout, PoE, Null0]]
    - Why: 2026-11-26, oob01 has no console login and no enable password. The design defers users (device-configuration.md line 80). Also: PoE off on unused ports, Null0 for unused 10.x on core01.
    - Done when: every network device asks for a login on the console, logs out idle sessions, and the design says so.
- [ ] [[Phase 1 Document Corrections]]
    - Why: the documents do not match the lab. Known: the port map labels the T470s as PAW-02. To verify: core01 is 48P, esxi01 has 8 NICs, the iLO name itc-uvy-ms-01, dc01 iDRAC placement.
    - Done when: the Phase 1 design documents match the devices after validation.
- [ ] [[Privileged Access Log 1.1 — break-glass from another workstation]]
    - Why: the PN52 is gone. The Wi-Fi-off rule assumed two workstations (H4), and no break-glass workstation exists (H2).
    - Proposal: Wi-Fi may stay on for console-only sessions and stays off for Gi0/8. The PAW end of the cable is the single control.
    - Done when: the log's rules work with one workstation, published as version 1.1.
- [ ] [[Reset All Servers to Baseline (normal change)]]
    - Why: servers and BMCs start Phase 2 from a known baseline (agreed 2026-10-08). Network devices keep their Phase 1 config.
    - Done when: every server and BMC is at baseline, with a change record.
- [ ] [[dc01 disk redundancy]]
    - Why: dc01 becomes the domain controller in Phase 2 and has no disk redundancy yet. Plan: 2x 300 GB SAS once a second 3.5" to 2.5" adapter is available.
    - Done when: dc01 runs on mirrored disks (RAID1) before AD is installed.
- [ ] [[Repo structure and navigation]]
    - Why: the Master Document has drifts D1-D12 (2026-11-26). Also covers README/CHANGELOG, Charter folders, an ADR home, and removing the Obsidian/ folder.
    - Done when: the Master Document matches git ls-files, and Obsidian/ is gone from the repo.


## Ready



## Priority



## WIP

- [ ] [[Apply Device Configuration 1.2 and re-run validation]]

## Review



## Done

- [ ] [[Change-Management SOP 1.1 — scope, lifecycle by type, approval evidence]] — Cancelled: superseded by lighter operating model (2026-11-06); PR #33 closed unmerged
- [ ] [[Retro Change Records — 10-02 disk swap, 10-06/07 rack work]]
- [ ] [[Operational Change Procedure 1.1 — backout choices and point of no return]]
- [ ] [[Operational Change Procedure]]
- [ ] [[Documentation Standard — approve and add document types]]
- [ ] [[PAW-01 Break-Glass End State (E4)]]
- [ ] [[Rack Inventory Verification]]
- [ ] [[Correct Asset Inventory and Project Structure sheet]] — Cancelled: premise invalid (inventory not in repo)
- [ ] [[Device Configuration Revision — management ACL and hardening]]
- [ ] [[Validation Plan Revision — V8 redesign]]
- [ ] [[Device Configuration Revision — bring-up deviations]]
- [ ] [[Phase 1 Network Fabric Bring-Up]]
- [ ] [[Validation and Handover]]
- [ ] [[Device Configuration and Bring-Up]]
- [ ] [[Routing and ACL Design]]
- [ ] [[VLAN and IP Address Plan]]
- [ ] [[Segmentation Design]]
- [ ] [[Physical Port Map Revision — WAN Topology and Lifeline Removal]]
- [ ] [[Physical Port Map and Rack Verification]]
- [ ] [[Update Master Document]]
- [ ] [[Repo bootstrap]]
- [ ] [[Write Project Charter]]
- [ ] [[Write Change-Management SOP]]
- [ ] [[Write Documentation Standard]]
- [ ] [[Populate docs/learning companions]]
- [ ] [[Write root README]]
- [ ] [[Write Phase 0 Learning Log entries]]
- [ ] [[Charter compliance pass and naming alignment]]




%% kanban:settings
```
{"kanban-plugin":"board"}
```
%%