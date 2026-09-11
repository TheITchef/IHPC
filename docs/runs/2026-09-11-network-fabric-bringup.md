# Network Fabric Bring-Up Run — records the first powered bring-up of the Phase 1 fabric and the validation results obtained at the bench.

Status: Draft · Version: 0.1 · Last updated: 2026-09-11 · Owner: Ioannis Mintzivyris

## 2. Overview

This document records a bench session, not a design. It is the evidence that the Phase 1 fabric was built on real hardware and behaved as the approved designs said it would.

On 2026-09-11 all three network devices were powered, verified against a clean baseline, configured per the approved Phase 1 designs, and tested as far as an empty fabric allows. The devices were administered by serial console throughout, from the PN52 over PuTTY at 9600 8-N-1. The PN52 was used for console access only; it is not a lab-access host and holds no network path into the fabric.

The session executed part of the validation plan written in Card 6. That plan states 27 tests with an empty result column. Roughly half were runnable in this session — every test that does not require a host on a test port or a vantage point outside the lab. The remainder stay open.

Results are recorded as found. Where a test failed, the failure is written down and the cause identified; no configuration was changed to make a test pass, and no test was rewritten after the fact to match what happened. One test (V8) was found to be badly conceived rather than the fabric being at fault, and is recorded as such.

## 3. Scope

### In scope

- The Phase 1 network fabric only: itc-uvy-core01, itc-uvy-rtr01, and itc-uvy-oob01.
- Baseline verification of each device before configuration, and the remediation required to reach a clean baseline.
- Application of the approved Phase 1 configuration to all three devices.
- The subset of the Card 6 validation plan runnable without hosts: config-state checks on all three devices, and the transit link tests.
- Deviations found between the approved design and what the hardware actually permits.

### Out of scope

- Hosts and their services. No server, workstation or controller was connected; the fabric remains empty.
- Tests requiring a laptop on a live access port (V12, V16, V21, V23) and the counters that depend on them (V22).
- The inbound-deny proof from outside the lab (V17) and its counter check (V18).
- Any revision of the approved design documents. Deviations are recorded here; correcting `device-configuration.md` is separate work.

## 4. Dependencies

- **Device Configuration and Bring-Up** (Approved 1.0) — the configuration applied to each device in this session. The document this run implements, and the one the recorded deviations correct.
- **Validation and Handover** (Approved 1.0) — the 27-test validation plan this run partially executes. Test identifiers used here refer to that document.
- **Physical Port Map** (Approved 1.2) — the port and cabling assignments the session followed, including the patch-panel path for the management uplink.
- **VLAN and IP Address Plan** (Approved 1.0) — the VLAN IDs, subnets and gateway addresses applied.
- **Routing and ACL Design** (Approved 1.0) — the routing, filtering and NAT posture the configuration implements and the tests check.

## 5. Deliverables

- **A configured and saved fabric** — all three devices holding the Phase 1 configuration in startup-config, with both fabric links live.
- **Executed validation results** — a result or a stated reason for every one of the 27 tests in the Card 6 plan.
- **A record of deviations** — the differences between the approved design and the fabric as built, each with its cause.
- **Bring-up observations** — behaviour encountered during the session that the design documents did not anticipate and that would cost time to rediscover.

## 6. Detailed content

### 6.1 Baseline verification and remediation

Each device was inspected before any configuration was applied. Two of the three were not as clean as they first appeared.

**itc-uvy-core01.** The prompt read `Switch>` and the running configuration held only IOS-XE defaults — no hostname, no SVIs, no routing, no access lists. The startup configuration was not present. The device was treated as clean and configured directly. Later in the session `show vlan brief` revealed three orphan VLANs (40 "Test-Staging", 50, 99) that the running configuration had not shown. All three were empty of member ports and were removed.

**itc-uvy-rtr01.** The device held a complete prior configuration under the same hostname: an older addressing scheme (a VLAN 10 SVI at 10.10.10.254/24), NAT for 10.10.10.0/24, an ACL-based stateless firewall, and in-band SSH restricted by access class. None of it matched the approved design. It was erased with `write erase` and reloaded. On reboot the device came up at factory default and immediately took a DHCP lease from the ISP on Gi8.

**itc-uvy-oob01.** Running configuration was clean and startup configuration was not present, but the VLAN database held four orphan VLANs (10 "MGMT", 30, 99 "OOB-MGMT", 999 "NATIVE"). All were empty and were removed before VLAN 10 was recreated under the project's naming.

The pattern across both switches is recorded as an observation in 6.5: the VLAN database is not part of the configuration that `write erase` clears.

### 6.2 Configuration outcome by device

All three devices were configured in the bring-up order the design specifies: core switch, then edge router, then management switch. Each was saved to startup-config before the next was started.

**itc-uvy-core01 — core switch.** Hostname applied; VLANs 10, 20 and 30 declared; `ip routing` enabled; the three segment gateways created at 10.10.0.1/24, 10.20.0.1/24 and 10.30.0.1/30. Gi1/0/46 placed in VLAN 30 as the transit port and Gi1/0/48 configured as a trunk allowing VLAN 10 only. The MGMT-IN access list was created with its enforcing deny and applied inbound on Vlan10; the two PAW permits were not applied, for the reason given under V11. The interface activation policy was applied in full: server ports placed in VLAN 20 and held shut with descriptions, prepared drops held shut with no VLAN, and every remaining port shut and labelled.

**itc-uvy-rtr01 — edge router.** Hostname applied. The transit address was placed on an SVI rather than the physical port, for the hardware reason recorded under V24 and in 6.4. NAT was configured with Gi8 as the outside interface, Vlan30 as the inside interface, and the NAT-SRV list naming the servers segment alone. The zone-based firewall was built policy-first — class-map, policy-map, zones, zone-pair — with interface zone-membership applied last, so that the default-deny became active only after the outbound policy existed. One static return route was added for the servers segment. The unused LAN ports Gi0 through Gi6 were shut.

**itc-uvy-oob01 — management switch.** Hostname applied and VLAN 10 declared. The uplink Gi0/9 was configured as an explicit trunk allowing VLAN 10 only, with DTP negotiation disabled. The controller ports and the PAW-01 port were placed in VLAN 10 and held shut; all remaining ports were shut and labelled.

### 6.3 Validation results

Test identifiers refer to the validation plan in **Validation and Handover**. Tests not run are marked NOT RUN with the reason. No result is inferred.

**Management switch (plan section 6.2)**

| ID | Result |
|----|--------|
| V1 | PASS — hostname `itc-uvy-oob01` set and confirmed. |
| V2 | PASS — after the orphan VLANs were cleared, only VLAN 10 (mgmt) is declared, alongside the default and legacy VLANs. |
| V3 | PASS — Gi0/1–3 (controllers) and Gi0/8 (PAW-01) in access mode, VLAN 10, administratively shut. Gi0/4–7 and Gi0/10 shut as unused. |
| V4 | PASS — Gi0/9 trunk, allowed VLAN list 10 only, connected at 1 Gbps full duplex. |

**Inter-VLAN routing (plan section 6.3)**

| ID | Result |
|----|--------|
| V5 | PASS — `ip routing` present and active. |
| V6 | AS FOUND — Vlan10 up/up once the management trunk was live; Vlan30 up/up once the transit link was live; Vlan20 up/down, having no live host port; Vlan1 down, unused by design. This is the expected empty-fabric state. |
| V7 | PARTIAL PASS — connected routes present for 10.10.0.0/24 and 10.30.0.0/30. 10.20.0.0/24 is absent because its interface line protocol is down; a connected route is installed only for an operational interface. Correct behaviour. |
| V8 | TEST DESIGN FLAW — see 6.4. The transit gateway replies. The management gateway does not: the ping is denied by MGMT-IN. The servers gateway does not: its interface is down. |
| V9 | PASS — the core console reaches 10.30.0.2 across the transit link. |

**Management default-deny (plan section 6.4)**

| ID | Result |
|----|--------|
| V10 | PASS — MGMT-IN applied inbound on Vlan10. |
| V11 | PARTIAL — the access list holds its enforcing deny only. The two PAW permits were deliberately not applied: they reference controller addresses that do not yet exist, and the approved design establishes them as documented intent rather than active enforcement at this interface. The protection of the segment is in place. |
| V12 | NOT RUN — requires a laptop on a live servers-segment access port. |
| V13 | PASS — during V8 the access list logged denied ICMP packets and its match counter incremented. The deny logs as designed. |

**Edge inbound-deny (plan section 6.5)**

| ID | Result |
|----|--------|
| V14 | PASS — the transit interface is in the inside zone and the WAN interface in the outside zone. One zone-pair exists, inside to outside; there is no outside-to-inside pair. |
| V15 | PASS — the outbound policy inspects TCP, UDP and ICMP; the default class drops. No inbound policy exists. |
| V16 | NOT RUN — requires a laptop on a live servers-segment access port. |
| V17 | NOT RUN — requires a vantage point outside the lab. |
| V18 | NOT RUN — depends on V17. |

**NAT scoping (plan section 6.6)**

| ID | Result |
|----|--------|
| V19 | PASS — the WAN interface is marked outside and the transit interface inside. |
| V20 | PASS — the NAT selection list names the servers segment alone. No other segment is present. |
| V21 | NOT RUN — requires a laptop on a live servers-segment access port. |
| V22 | NOT RUN — depends on V23. |
| V23 | NOT RUN — requires a laptop on a live management-segment access port. |

**Transit link (plan section 6.7)**

| ID | Result |
|----|--------|
| V24 | FAIL — hardware limitation. The edge router's LAN ports cannot be converted to routed ports. See 6.4. |
| V25 | PASS — both ends of the transit subnet addressed as designed, both up. |
| V26 | PASS — the link carries traffic in both directions, confirmed from each device in turn. |
| V27 | PASS — the static return route for the servers segment is present. No route to the management segment exists on the edge, confirming its isolation. |

### 6.4 Findings requiring a change

**V24 — the edge router's transit port cannot be a routed port.**

The approved configuration converts the router's transit interface to a routed port and applies the transit address to it directly. The hardware does not permit this. The router's LAN-side ports are an integrated Layer 2 switch; the command to convert one to a routed port is not available, and the platform offers only Layer 2 sub-options in its place.

The transit address was therefore placed on a VLAN interface, with the physical port configured as an access port in that VLAN. The result is electrically identical — the same address at the same end of the same point-to-point subnet — and the transit link was subsequently proven working in both directions. The approved design names this as the alternative pattern; it is now the actual one.

The NAT inside marking follows the address. The approved configuration places it on the physical port; it is applied to the VLAN interface instead, because network address translation operates where the Layer 3 address lives.

**V8 — the test is at fault, not the fabric.**

V8 pings each segment gateway from the core switch console. For the management gateway this cannot succeed while the management access list is applied. Traffic sourced from the switch itself arrives inbound on the management interface, where the deny-only rule drops it and logs the drop. The access list behaved exactly as designed; the test assumed a reachability check that the management posture is built to prevent.

The result is recorded as found. No configuration was changed in response, and the test was not quietly rewritten to match. V8 needs redesigning in the next revision of the validation plan — either sourcing the ping from a host inside the segment being tested, or scoping the test to the gateways that carry no inbound filter and validating the management gateway another way.

### 6.5 Deviations from the approved design

Each item below is a difference between the fabric as built and **Device Configuration and Bring-Up** (Approved 1.0). Correcting that document is separate work.

1. **The transit address sits on a VLAN interface, not the physical port** — the hardware limitation recorded under V24.
2. **The NAT inside marking sits on the same VLAN interface** — following from the above.
3. **PAW-02 no longer exists.** The sole privileged access workstation is PAW-01. The approved configuration names PAW-02 in the management access list permits, in the management addressing-lane table, and in the management switch port description. All three should read PAW-01.
4. **DTP negotiation disabled on the management uplink.** Not in the approved configuration. Added after the uplink was observed forming a trunk automatically from its default negotiating state. Disabling negotiation removes the path by which a connected device can request a trunk.
5. **Two small router settings not in the approved configuration** — the legacy network-boot behaviour was disabled, and hostname resolution was turned off. Both were added during bring-up, the first because the setting appeared after the erase, the second to stop the router attempting to resolve names supplied by the ISP.

### 6.6 Observations

- **Erasing the configuration does not clear the VLAN database.** On both switches the running configuration was clean while orphan VLANs remained. The database is held separately in flash. A genuine factory reset requires erasing the configuration, deleting the VLAN database file, and reloading.
- **The ISP's DHCP lease supplies more than an address.** After the router was erased it took a lease that set its hostname, configured an external time server, and installed a default route and a host route. The hostname and time server were removed. The default route is required and remains.
- **The WAN link was live before any configuration existed.** The router pulled its lease during boot. The interval between that and the firewall being in place is worth keeping short.
- **Firewall bring-up order matters.** The approved configuration lists the zones and their interface membership before the policy. Applied in that order, the default-deny would become active before any permitting policy existed. The firewall was built in the reverse order — matching rule, policy, zones, zone-pair, and interface membership last — so that the permits existed before the deny took effect. This is bring-up sequencing guidance the design document does not carry.

### 6.7 State at end of session

All three devices hold the Phase 1 configuration in startup-config. Both fabric links are live: the transit link between the core switch and the edge router, and the management trunk between the core switch and the management switch. The management and transit gateways are up; the servers gateway waits for its first host.

No host is connected. The fabric is routed, filtered, documented and empty, as Phase 1 intends.

## 7. Acceptance criteria

- Every one of the 27 tests in the validation plan has either a result or a stated reason for not being run. Nothing is left ambiguous and no result is inferred.
- Failures are recorded as found. No configuration was changed to make a test pass, and no test was reworded after the fact to match the outcome.
- Every deviation between the built fabric and the approved design is recorded, with its cause.
- Baseline findings are recorded, including residual state that the initial inspection did not reveal.
- The end state of the fabric is stated, including what remains untested and why.

## 8. References

- **Device Configuration and Bring-Up** — the configuration applied in this session and the document the recorded deviations correct.
- **Validation and Handover** — the validation plan this run partially executes; the source of all test identifiers used here.
- **Physical Port Map** — the port and cabling assignments followed.
- **VLAN and IP Address Plan** — the VLAN IDs, subnets and gateway addresses applied.
- **Routing and ACL Design** — the routing, filtering and NAT posture implemented and tested.
- **Master Document** — where this document sits in the repository.