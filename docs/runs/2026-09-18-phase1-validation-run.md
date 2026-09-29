# Phase 1 Validation Run — records the second bench session, which ran the host-dependent tests of the Phase 1 validation plan.

**Status:** Approved · **Version:** 1.0 · **Last updated:** 2026-09-29 · **Owner:** Ioannis Mintzivyris

## 2. Overview

This document records a bench session, not a design. On 2026-09-18 the Phase 1 fabric was tested with a host on the wire and a vantage point outside the lab. These were the tests the first run (2026-09-11) could not execute.

**This record was written after the fact.** The validation plan requires a dated run record for every execution. None was written on the day. It was written on 2026-09-29 from notes taken during the session. Where the notes do not record a detail, this document says so rather than filling it in.

The session found two errors in the approved design and one badly conceived test. One error was fixed at the bench. The other was not. Results are recorded as found.

## 3. Scope

### In scope

- The tests in the validation plan that need a host on a test port or an external vantage point: V6, V7, V12, V13, V16, V17, V18, V21, V22, V23, V28.
- A reinterpretation of the 2026-09-11 result for V8, based on evidence from this session.
- Findings that require a change to the approved design, and deviations from the approved test method.

### Out of scope

- The configuration-state tests already run on 2026-09-11 (V1–V5, V9–V11, V14–V15, V19–V20, V24–V27). They were not re-run.
- Correcting the design documents. The findings are recorded here; the corrections are separate work.

## 4. Dependencies

- **Validation and Handover** (Approved 1.1) — the plan this session executes. Test identifiers refer to that version.
- **Device Configuration and Bring-Up** (Approved 1.1) — the configuration under test.
- **Routing and ACL Design** (Approved 1.0) — the management posture the tests check.
- **Network Fabric Bring-Up Run (2026-09-11)** — the first execution of the plan, which this session completes.

## 5. Deliverables

- **Executed results** — a result or a stated reason for every test in the plan.
- **Findings** — the design errors and test faults found, each with its evidence.
- **Deviations** — where the session departed from the approved method or process.
- **End state** — the configuration the fabric was left in.

## 6. Detailed content

### 6.1 Session setup

- **Date:** 2026-09-18, morning, at the rack.
- **Test host:** the ASUS PN52, Windows 11, with a USB-C Ethernet adapter. It stood in for hosts that do not exist yet. It is not a lab device.
- **Test port:** Gi1/0/40 on the core switch. An unused port, described `TEST HOST temporary validation port`, and assigned to whichever VLAN was under test. It was reverted afterwards.
- **Test host addresses:** 10.20.0.50 on VLAN 20; 10.10.0.60 on VLAN 10.
- **External vantage:** an Android phone on cellular data, Wi-Fi off.
- **Source-bound tests:** the test host's Wi-Fi stayed connected (see Deviation 1). Every ping was therefore forced out of the Ethernet adapter with `ping -S <source> <destination>`.

### 6.2 Validation results

Test identifiers refer to **Validation and Handover** version 1.1. No result is inferred.

**Management switch (plan section 6.2)**

| ID | Result |
|----|--------|
| V1–V4 | NOT RUN — executed 2026-09-11. |

**Inter-VLAN routing (plan section 6.3)**

| ID | Result |
|----|--------|
| V5 | NOT RUN — executed 2026-09-11. |
| V6 | PASS — all three SVIs up/up. Vlan20 came up when the test host joined VLAN 20. This was the first time all three gateways were live together. |
| V7 | PASS — connected routes present for 10.10.0.0/24, 10.20.0.0/24 and 10.30.0.0/30. |
| V8 | NOT RUN — the 2026-09-11 result is reinterpreted in Finding 1. |
| V9 | NOT RUN — executed 2026-09-11. |
| V28 | FAIL — from VLAN 10, the test host could not ping its own gateway, 10.10.0.1. MGMT-IN denied it and logged the drop. See Finding 1. |

**Management default-deny (plan section 6.4)**

| ID | Result |
|----|--------|
| V10–V11 | NOT RUN — executed 2026-09-11. |
| V12 | FAIL — from VLAN 20, the test host pinged the management gateway 10.10.0.1 and got four replies. The management segment's gateway was reachable from the Servers segment. See Finding 1. |
| V13 | INVALID — depends on the MGMT-IN counter moving after V12. The counter stayed at zero, because the traffic never passed the interface direction the rule was applied to. |

**Edge inbound-deny (plan section 6.5)**

| ID | Result |
|----|--------|
| V14–V15 | NOT RUN — executed 2026-09-11. |
| V16 | FAIL, then PASS after a bench fix — from VLAN 20, a ping to 8.8.8.8 first returned "destination host unreachable" from the gateway. After the default route was added (Finding 2), the ping succeeded and replies returned through the firewall. |
| V17 | PASS — from the phone on cellular, HTTP and HTTPS to the WAN address both failed. A scan of ports 1–2000 found none open. See Observation 4. |
| V18 | UNRUNNABLE AS WRITTEN — the counters the test checks do not record inbound drops. See Finding 3. |

**NAT scoping (plan section 6.6)**

| ID | Result |
|----|--------|
| V19–V20 | NOT RUN — executed 2026-09-11. |
| V21 | PASS — with the test host generating traffic from VLAN 20, the router showed two translations: 10.20.0.50 to the public WAN address, on distinct ports. |
| V22 | PARTIAL — no 10.10.0.0/24 source appeared in the translations. But nothing on VLAN 10 was generating traffic at that moment, so this proves little alone. V23 covers the claim better. |
| V23 | PASS — from VLAN 10, a ping to 8.8.8.8 failed with "destination host unreachable". The result is over-determined: the access list, the absent NAT entry, and the absent return route each block it alone. |

**Transit link (plan section 6.7)**

| ID | Result |
|----|--------|
| V24–V27 | NOT RUN — executed 2026-09-11. |

### 6.3 Findings requiring a change

**Finding 1 — the management access list filtered the wrong direction. Not fixed in this session.**

The design says nothing from another segment reaches management. As built, MGMT-IN (`deny ip any any log`) was applied **inbound** on Vlan10. Inbound on an SVI means traffic arriving from hosts *inside* that VLAN. So the rule blocked management's own traffic and left the segment open from other segments.

Evidence:

- From VLAN 20, the test host pinged 10.10.0.1: four replies, TTL 254, 2 ms. A real routed path, not a Wi-Fi side path.
- The MGMT-IN counter stayed at zero.
- From VLAN 10, the test host could not ping its own gateway. The log showed `%SEC-6-IPACCESSLOGP: list MGMT-IN denied icmp 10.10.0.60 -> 10.10.0.1`, plus denied SSDP, mDNS and IGMP from the same host.

A fix was tried at the bench: MGMT-IN applied outbound on Vlan10 as well. The platform accepted it, but it could not be validated. No host existed inside VLAN 10 to act as a target, and a ping to an absent address stops at ARP, so the counter does not move. The change was reverted. See Deviation 2.

A second gap surfaced: traffic addressed to the gateway itself (10.10.0.1) ends at the switch. It is never forwarded into VLAN 10, so no ACL on Vlan10, in either direction, protects the gateway address.

**The 2026-09-11 result for V8 is reinterpreted.** That day, the core console's own ping to 10.10.0.1 was denied and logged by MGMT-IN. It was read as the access list working correctly. It was the same inverted control, catching traffic the switch sourced itself.

Intent settled during the session: management is fully isolated. No routed traffic in either direction. Exceptions are considered only if a real need appears.

**Finding 2 — the core switch had no default route. Fixed at the bench and verified.**

The design gave the edge a return route to 10.20.0.0/24 but gave the core no route toward the edge. Internal segments could reach each other; nothing could reach the internet.

Evidence: "Gateway of last resort is not set" on the core. From VLAN 20, a ping to 8.8.8.8 got "destination host unreachable" from its own gateway.

Fix: `ip route 0.0.0.0 0.0.0.0 10.30.0.2` on the core. Verified: gateway of last resort 10.30.0.2, outbound ping succeeded, and NAT translations appeared on the router. Saved to startup-config. The route was in no approved document at the time. See Deviation 2.

**Finding 3 — V18 cannot run as written.**

V18 checks the firewall's drop counters after V17. No such counters exist for this traffic. The edge's inbound deny works by the absence of an outside-to-inside zone-pair. Inbound traffic hits the firewall's default deny, above any policy-map, so nothing counts it.

Evidence: `show policy-map type inspect zone-pair` showed every counter at zero, including class-default. That policy belongs to the outbound pair (IN-OUT) and never sees inbound traffic. `show zone-pair security` confirmed IN-OUT is the only pair.

The firewall is not at fault. The test assumed the deny would leave a trace.

### 6.4 Deviations

**Deviation 1 — the test host was dual-homed.** The PN52's Wi-Fi stayed connected to the home network while its Ethernet sat in VLAN 20, and later in VLAN 10. For that time, one machine was connected to both the untrusted home network and the Tier 0 segment. The validation plan names the PN52 as not a lab-access host. Source-bound pings kept the test traffic on the right path, but they do not remove the exposure. No harm was observed. Future sessions must disconnect every other network on the test host while it is on a lab port.

**Deviation 2 — configuration was changed during a validation session, outside the change-management process.** The default route was added and saved. MGMT-IN was applied outbound and then reverted. Neither went through a change request. The run of 2026-09-11 changed nothing to make a test pass; this session did. The default route stays, and is now documented in Device Configuration and Bring-Up version 1.2.

**Deviation 3 — the VLAN 10 tests used a port on the core switch.** The plan places the VLAN 10 test port on the management switch. The session used Gi1/0/40 on the core for both VLANs. The results are unaffected: the port was in VLAN 10, and the traffic met the same Vlan10 interface.

**Deviation 4 — this record was written late.** See the Overview.

### 6.5 Observations

1. **A ping to an absent host does not test an ACL.** It stops at ARP (the address lookup before sending), so no packet reaches the rule and its counter stays at zero. Testing a deny toward a segment needs a real host inside that segment.
2. **Traffic to a gateway's own address is not routed into its VLAN.** It ends at the switch. ACLs on that VLAN's interface do not see it when it arrives from another segment.
3. **The zone-based firewall's default deny leaves no counter.** Proof of the inbound deny comes from the external test (V17) and the zone-pair configuration (V14), not from a counter.
4. **The carrier filters some ports.** In the V17 scan, ports 137–142 and 445–463 (NetBIOS and SMB) showed as "blocked" rather than "closed". This is filtering on the mobile network, not the lab firewall.
5. **A Windows host is chatty.** Alongside the test pings, MGMT-IN logged denied SSDP, mDNS and IGMP from the test host. Any Windows host on a filtered segment will generate this.

### 6.6 State at end of session

- Test port Gi1/0/40 reverted: description `UNUSED held shut`, no VLAN, administratively shut.
- The default route is in place and saved on the core.
- MGMT-IN remains applied inbound only on Vlan10. The management segment is not protected from other segments.
- The test host is disconnected.

## 7. Acceptance criteria

- Every test in the validation plan has a result or a stated reason for not being run in this session.
- Failures are recorded as found, with their evidence.
- Every configuration change made during the session is recorded, including the one reverted.
- Every departure from the approved method or process is recorded as a deviation.
- The end state of the fabric is stated, including what remains unprotected.
- The record states that it was written after the fact, and does not fill gaps from memory.

## 8. References

- **Validation and Handover** — the plan executed; the source of all test identifiers.
- **Device Configuration and Bring-Up** — the configuration under test, and the document the findings correct.
- **Routing and ACL Design** — the management posture tested.
- **Network Fabric Bring-Up Run (2026-09-11)** — the first execution of the plan.
- **Change-Management SOP** — the process Deviation 2 departed from.
- **Master Document** — where this document sits in the repository.