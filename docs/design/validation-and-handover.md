# Validation and Handover — defines how the Phase 1 fabric is proven correct, and the state it hands to Phase 2.
Status: Approved · Version: 1.1 · Last updated: 2026-09-17 · Owner: Ioannis Mintzivyris

## 2. Overview

This document closes Phase 1. It does two things: it defines how the built fabric is proven correct, and it records the state Phase 1 hands to Phase 2.

The fabric was designed and configured across the earlier Phase 1 cards — physical port map, segmentation, VLAN and IP plan, routing and ACLs, and device configuration. This document does not repeat those designs; it verifies them. Each design claim (routing works where allowed, filtering holds where required, NAT is scoped correctly) becomes a test with an expected result.

Design-now, execute-later. This document was written while the lab was powered off and worked on remotely, so it states every test and its expected result without running any of them. That split still governs how it is used: the plan defines what correct looks like, and the bench decides whether the fabric matches. Results are not recorded in this document — each execution produces its own dated run record, so the plan stays a stable statement of intent rather than a log.

The handover records what Phase 1 leaves behind — a routed, filtered, documented, empty fabric — and what Phase 2 inherits. It also closes the two build-time loose ends parked for this card: the temporary lifeline into the core switch and the break-glass access host.

**Version 1.1 corrects the plan against the first bench run.** The plan was written before the fabric existed. Running it found one test that could not pass on a correct fabric, and several places where the plan described a configuration the hardware does not permit. Those corrections are folded into the sections below. The run record holds the evidence.

## 3. Scope

### In scope

- Validation of the Phase 1 fabric as built: inter-VLAN routing where the design permits it, the two default-deny boundaries (management inbound, edge inbound), NAT scoped to servers only, and the transit link carrying traffic.
- The Phase 1 handover: the state the fabric is left in and what Phase 2 inherits.
- Closure of the two build-time loose ends parked for this card: the temporary lifeline into the core switch, and the break-glass access host.

### Out of scope

- Test results. This document is the plan; each execution is recorded in its own dated run record under `docs/runs/`.
- Application or service testing. The fabric is empty — no endpoints or servers exist yet — so there is nothing above the network layer to test.
- Controller reachability. The management ACL still carries `<controllers>` placeholders; real addresses are assigned when hardware is introduced. See Handover.
- The parked in-document deferrals — the management addressing-lane scheme and the oob01 management SVI. Both are future work, not Phase 1 state. See Handover.

## 4. Dependencies

This card validates the Phase 1 fabric, so it depends on every approved Phase 1 design. It adds no new design; it proves the ones already settled. Each document below contributes the claim this card tests against it.

- **Physical Port Map** (Approved 1.2) — which device interface lands on which port. The tests reference real ports when a link is exercised; this is the source for them.
- **Segmentation Design** (Approved 1.0) — the three segments and the trust boundaries between them. Defines what "correctly isolated" means — the boundaries the deny tests prove hold.
- **VLAN and IP Address Plan** (Approved 1.0) — the VLAN IDs, subnets, and gateway addresses. The source for every concrete address a test uses as a source or destination.
- **Routing and ACL Design** (Approved 1.0) — the routing responsibilities, the transit link, the static return route, and the management flow matrix. Defines what "routes where allowed, denies where required" means — the expected results the routing, ACL, and NAT tests check against.
- **Device Configuration and Bring-Up** (Approved 1.1) — the configuration actually applied to each device. The fabric under test; every test proves a line of this document does what it was written to do.

## 5. Deliverables

- **A validation plan** — the tests that prove the Phase 1 fabric correct, grouped by the claim each proves: inter-VLAN routing where the design permits it, the management default-deny, the edge inbound-deny, NAT scoped to servers only, and the transit link carrying traffic. Each test states a method and the result that means the design is correct.
- **A handover** — the state Phase 1 leaves behind (a routed, filtered, documented, empty fabric) and what Phase 2 inherits.
- **Two loose-end closures** — the recorded decisions that retire the two build-time exceptions parked for this card: the temporary lifeline into the core switch, and the break-glass access host.

The plan is a standing statement of what correct looks like. It is executed at the bench as often as the fabric changes, and each execution is recorded separately.

## 6. Detailed content

### 6.1 Validation approach

This document states the tests; the bench runs them. Each test names what it proves, the exact command to run and where to run it from, and the result that means the design is correct. Nothing is left for the operator to derive at the console.

Every test is one row in a table with five columns:

| Column | Meaning |
|---|---|
| **ID** | A stable identifier for the test (V1, V2 …), so a run record, the learning companion, and any write-up can reference a test by name. |
| **Test** | What is being proven, in one line. |
| **Method** | The exact command to run, and where it is run from. Self-contained, so the test is runnable at the bench without re-deriving it. |
| **Expected** | The result that means the design is correct, together with the connections required for the test to be meaningful. |
| **Actual (bench)** | Where the operator notes what happened in the moment. |

A test **passes** when Actual matches Expected. A mismatch is not quietly corrected in this document — it is recorded as-found, and the fix is handled as its own change, so neither the plan nor the record is edited to flatter the result. V8 is a worked example: it failed at the first bench run, the failure was recorded, and the test was redesigned afterwards as its own change rather than being silently reworded.

**A note on where results live.** The Actual column is where a bench operator notes what happened in the moment. The authoritative record of any execution is its dated run record under `docs/runs/`, which carries the full result for every test, the reason for any test not run, and anything found that the plan did not anticipate.

The tests are grouped by the claim each group proves. Six groups follow: the standalone management-switch check, then one per boundary the design makes a claim about — inter-VLAN routing, the management default-deny, the edge inbound-deny, NAT scoping, and the transit link.

**A note on the empty fabric.** Phase 1 ends with no hosts connected — a routed, filtered, empty fabric. Several tests therefore run from a device console (the core switch, the edge router) or from a technician's laptop connected to a test port for the duration of the test, not from a permanent host. Where a test needs a source that does not exist yet, the Method says so and names the stand-in. No test assumes a server or workstation that Phase 1 never installed.

### 6.1.1 Bench prerequisites — power tiers

The tests are written to run top to bottom in power order. Set the bench up once, in these tiers, and each group becomes runnable in turn:

| Tier | What is powered | Groups runnable |
|---|---|---|
| 1 — Management switch alone | oob01 only (core and edge **down**) | 6.2 (V1–V4) |
| 2 — Core added | core switch powered | 6.3 config checks (V5–V7), 6.4 config checks (V10–V11) |
| 3 — Core + edge | both powered, transit link live | 6.5, 6.6, 6.7, and every traffic test across 6.3–6.4 |
| + laptop on a test port | a technician laptop on a live VLAN 20 or VLAN 10 access port | the host-sourced tests (V12, V16, V21, V23, V28) |
| + external vantage | a host outside the lab (internet side or cellular) | the inbound-deny proof (V17) |

The management switch (oob01) is checked cold, first, because it needs nothing else. The traffic tests come last, once the core and edge are both up and the transit link between them is live.

### 6.2 Management switch (oob01) — standalone config check

*Proves the management switch is built per Card 5. oob01 is a Layer 2 access switch with no IP address in Phase 1, so it cannot originate a Layer 3 test — but its own configuration can be fully verified from its console alone.*

**Timing:** these tests run from oob01's console and need nothing else powered. Run them with the core and edge **down**, as a standalone check of the management switch before the routed fabric is brought up. Powering core and edge changes none of these results.

| ID | Test | Method | Expected | Actual (bench) |
|----|------|--------|----------|----------------|
| V1 | Hostname is set | `show run \| include hostname` on oob01 | `hostname itc-uvy-oob01`.<br><br>*REQUIRES: oob01 powered and console access. Core and edge may be down.* | |
| V2 | Only VLAN 10 is declared | `show vlan brief` on oob01 | VLAN 10 (mgmt) present; no other user VLAN declared. Note that the VLAN database survives a configuration erase, so this checks `vlan.dat` as much as the configuration.<br><br>*REQUIRES: oob01 powered and console access. Core and edge may be down.* | |
| V3 | Access ports are in VLAN 10 and shut | `show run interface Gi0/1` … `Gi0/8` on oob01 | Controller and PAW-01 ports in access mode, VLAN 10, administratively shut.<br><br>*REQUIRES: oob01 powered and console access. Core and edge may be down.* | |
| V4 | Trunk uplink is explicit and allows VLAN 10 only | `show interfaces trunk` and `show run interface Gi0/9` on oob01 | Gi0/9 administratively trunk, allowed VLAN list = 10 only, `switchport nonegotiate` present, no shutdown.<br><br>*REQUIRES: oob01 powered and console access. Core and edge may be down. (Whether the trunk shows as operationally up depends on the core end being up — the configuration is verifiable regardless.)* | |

**Note on oob01 and Layer 3 tests.** oob01 holds no IP address in Phase 1 (its management SVI is deferred to the in-band-management work). Console access to oob01 therefore verifies oob01's own build only — it cannot be used as the source of a ping or any inter-VLAN test. Those originate from the core console, the edge console, or a laptop with an address on a test port, as the later groups specify.

### 6.3 Inter-VLAN routing

*Proves the core switch is routing and each segment gateway is live and reachable at its own address. Host-to-host inter-VLAN flows are validated when hosts exist; Phase 1 proves the routing plane itself.*

| ID | Test | Method | Expected | Actual (bench) |
|----|------|--------|----------|----------------|
| V5 | Layer 3 forwarding is enabled | `show run \| include ip routing` on the core switch | `ip routing` present; switch is routing.<br><br>*REQUIRES: core switch powered; no live connection needed.* | |
| V6 | All three SVIs are up | `show ip interface brief \| include Vlan` on the core switch | Vlan10, Vlan20, Vlan30 all `up/up`.<br><br>*REQUIRES: core switch powered; transit port live for Vlan30; management trunk live for Vlan10; a laptop on a VLAN 20 test port for Vlan20.* | |
| V7 | Connected segments are in the routing table | `show ip route connected` on the core switch | 10.10.0.0/24, 10.20.0.0/24, 10.30.0.0/30 all present as connected.<br><br>*REQUIRES: core switch powered; a segment's SVI must be up for its route to appear.* | |
| V8 | The unfiltered gateways answer at their own address | From the core console, `ping 10.20.0.1` and `ping 10.30.0.1` | Both reply. The management gateway (10.10.0.1) is deliberately excluded — see the note below — and is covered by V28.<br><br>*REQUIRES: core switch powered; the SVI being pinged must be up (see V6).* | |
| V9 | Core reaches the edge across transit | From the core console, `ping 10.30.0.2` | Reply from the edge router.<br><br>*REQUIRES: core switch and edge router powered; transit link live at both ends.* | |
| V28 | The management gateway answers from inside its own segment | Laptop on a VLAN 10 test port (address in 10.10.0.0/24, gateway .1); `ping 10.10.0.1` | Reply. The ping is switched at Layer 2 within VLAN 10 and never crosses the routed boundary, so MGMT-IN does not see it.<br><br>*REQUIRES: core switch powered; laptop connected to a live VLAN 10 access port on the management switch.* | |

Note: V6 records the SVI state as found at the bench. A host-segment SVI (Vlan10, Vlan20) holds its line protocol down until a port in that VLAN is live, so the state depends on what is physically connected when the test runs. The same dependency governs V7: a segment whose SVI is down has no connected route, and its absence from the routing table is correct rather than a fault.

**Why the management gateway is tested separately.** The obvious way to check a gateway is to ping it from the switch that hosts it. For Vlan10 that cannot work: traffic sourced from the switch itself arrives inbound on the Vlan10 interface, where MGMT-IN drops it and logs the drop. The access list is doing exactly what it was written to do, so the test would report a failure for a fabric that is correct.

This was found at the first bench run (2026-09-11), where the original form of V8 failed for precisely this reason. V8 is therefore scoped to the gateways that carry no inbound filter, and the management gateway moves to V28, sourced from inside its own segment where the access list does not apply.

Until a laptop or host exists on the management segment, V28 cannot run and the management gateway is evidenced only by its interface state in V6 — up, addressed, with its connected route present in V7. That is weaker than a reply, and the plan says so rather than implying the gateway is proven.

V28 is numbered out of sequence because it was added after the plan's first execution. Test identifiers are stable once a run record has cited them; renumbering would invalidate the references in that record.

### 6.4 Management default-deny

*Proves the management segment rejects traffic routed in from other segments. This is the ACL's real enforcement — the routed boundary. Intra-VLAN enforcement is deferred (see Device Configuration) and is not tested here.*

| ID | Test | Method | Expected | Actual (bench) |
|----|------|--------|----------|----------------|
| V10 | The ACL is applied inbound on the management gateway | `show ip interface Vlan10 \| include access list` on the core switch | `MGMT-IN` applied inbound.<br><br>*REQUIRES: core switch powered; no live connection needed.* | |
| V11 | The ACL matches the design | `show ip access-lists MGMT-IN` on the core switch | `deny ip any any log` present. The two PAW-01 permits (HTTPS and ICMP) are present only once controller addresses have been assigned from the .20–.49 lane; until then the enforcing deny stands alone, by decision rather than omission.<br><br>*REQUIRES: core switch powered; no live connection needed.* | |
| V12 | Servers segment cannot route into management | Laptop on a VLAN 20 test port (address in 10.20.0.0/24, gateway .1); `ping 10.10.0.1`, then `ping 10.10.0.10` | `ping 10.10.0.1` fails — dropped by the deny at the routed boundary (the true test). `ping 10.10.0.10` also fails, but only proves a drop once PAW-01 is connected — until then it fails for lack of a host and is not conclusive.<br><br>*REQUIRES: core switch powered; laptop connected to a live VLAN 20 access port.* | |
| V13 | The deny is logging | After V12, `show ip access-lists MGMT-IN` on the core switch | Match counter on `deny ip any any log` has incremented.<br><br>*REQUIRES: V12 run first (the traffic the counter records).* | |

### 6.5 Edge inbound-deny

*Proves the edge refuses internet-initiated traffic (enforced by the absence of an outside-to-inside policy) while permitting outbound sessions and their stateful replies.*

| ID | Test | Method | Expected | Actual (bench) |
|----|------|--------|----------|----------------|
| V14 | Interfaces are zoned correctly | `show zone security` and `show zone-pair security` on the edge router | Vlan30 in INSIDE, Gi8 in OUTSIDE; one zone-pair IN-OUT (inside→outside) only; no outside→inside pair. The inside zone member is the VLAN interface, not Gi7, because zone membership follows the Layer 3 interface.<br><br>*REQUIRES: edge router powered; no live connection needed.* | |
| V15 | Only outbound is inspected | `show policy-map type inspect zone-pair` on the edge router | INSIDE-TO-OUTSIDE inspects tcp/udp/icmp; class-default drops; no inbound policy exists.<br><br>*REQUIRES: edge router powered; no live connection needed.* | |
| V16 | Outbound session works and returns | Laptop on a VLAN 20 test port (gateway .1); `ping 8.8.8.8`, then browse to an HTTPS site | Both succeed — replies return via stateful inspection.<br><br>*REQUIRES: full fabric powered (core + edge); WAN link up with ISP DHCP lease; laptop on a live VLAN 20 access port.* | |
| V17 | Inbound is refused | From an external host, attempt a connection to the WAN address (`curl` to the public address, or an external port scan) | Connection refused or times out — no session established.<br><br>*REQUIRES: edge router powered and WAN link up; an external vantage point outside the lab (a host on the internet side, or a phone on cellular) to originate the inbound attempt; the current ISP-assigned WAN address noted.* | |
| V18 | Inbound attempts are dropped by default | After V17, `show policy-map type inspect zone-pair session` and the drop counters on the edge | Drops recorded on class-default; no inbound session created.<br><br>*REQUIRES: V17 run first (the inbound traffic the counter records).* | |

### 6.6 NAT scoping

*Proves NAT translates the Servers segment only. VLAN 20 shares one public address outbound; management is never translated and has no outbound path.*

| ID | Test | Method | Expected | Actual (bench) |
|----|------|--------|----------|----------------|
| V19 | NAT inside/outside are on the right interfaces | `show ip nat statistics` on the edge router | Gi8 marked outside, Vlan30 marked inside. The inside marking sits on the VLAN interface, not Gi7, because NAT operates where the Layer 3 address lives.<br><br>*REQUIRES: edge router powered; no live connection needed.* | |
| V20 | The NAT ACL names servers only | `show ip access-lists NAT-SRV` on the edge router | Permits 10.20.0.0/24 only; no other segment present.<br><br>*REQUIRES: edge router powered; no live connection needed.* | |
| V21 | Servers traffic is translated | Laptop on a VLAN 20 test port; generate outbound (`ping 8.8.8.8`); on the edge `show ip nat translations` | Translation entries appear for the VLAN 20 source behind the WAN address (PAT/overload).<br><br>*REQUIRES: full fabric powered (core + edge); WAN link up with ISP DHCP lease; laptop on a live VLAN 20 access port.* | |
| V22 | Management is not translated | `show ip nat translations` on the edge while management traffic is attempted (from the V23 laptop) | No translation entry for any 10.10.0.0/24 source.<br><br>*REQUIRES: run alongside V23 (the management traffic that would show up if it were wrongly translated).* | |
| V23 | Management has no outbound path | Laptop on a VLAN 10 test port; `ping 8.8.8.8` | Fails — management cannot reach the internet. This failure is over-determined: no NAT entry, no return route, and the management ACL all block it independently. It proves the end-to-end result (management is isolated from the internet), not NAT scoping alone; V20 and V22 isolate the NAT half.<br><br>*REQUIRES: full fabric powered; laptop on a live VLAN 10 access port.* | |

### 6.7 Transit link

*Proves the core–edge transit link is up and passing traffic — the single Layer 3 path between the routing spine and the edge. Isolates the link so a downstream failure can be told apart from a transit failure.*

| ID | Test | Method | Expected | Actual (bench) |
|----|------|--------|----------------|----------------|
| V24 | The router transit port carries the transit VLAN | `show run interface Gi7` on the edge router | Gi7 is an access port in VLAN 30. The transit address sits on `interface Vlan30`, not on the physical port — the 891F's LAN ports are Layer 2 only and cannot be converted to routed ports (see Device Configuration 1.1).<br><br>*REQUIRES: edge router powered; no live connection needed.* | |
| V25 | Both ends are addressed per the /30 | Core: `show ip interface brief \| include Vlan30`; edge: `show ip interface brief \| include Vlan30` | Core Vlan30 = 10.30.0.1, edge Vlan30 = 10.30.0.2, both up.<br><br>*REQUIRES: both devices powered; transit link cabled.* | |
| V26 | The link passes traffic both ways | Core console `ping 10.30.0.2`; edge console `ping 10.30.0.1` | Both reply — the link carries traffic in both directions.<br><br>*REQUIRES: both devices powered; transit link live at both ends.* | |
| V27 | The edge's return route points across transit | `show ip route static` on the edge router | Static route 10.20.0.0/24 → 10.30.0.1 present, and no route to 10.10.0.0/24. The routing table also carries the default route the ISP's DHCP lease installs, and may carry host routes belonging to the ISP; those are expected and not part of this design.<br><br>*REQUIRES: edge router powered; no live connection needed.* | |

### 6.8 Phase 1 handover state

**What Phase 1 delivered.** A routed, filtered, documented, empty fabric. Three network devices are configured and brought up: the core switch holds the routing for the lab and the three segment gateways; the edge router provides the internet path, translates the Servers segment outbound, and refuses everything inbound; the management switch presents the management segment at Layer 2. Three segments are defined — management, servers, transit — with the trust boundaries between them enforced: management is default-deny from the routed side, and nothing initiates into the lab from the internet. The fabric carries no hosts; the only live links are the fabric connections between the three network devices.

**What Phase 2 inherits.** A working foundation and a set of known conditions on it. The foundation: a live routing spine on the core, an internet path through the edge, and three addressed segments ready to be populated. The conditions: no hosts exist yet, so every host-facing port is held shut awaiting its device; some design elements are deliberately deferred rather than missing. The deferrals and placeholders are recorded where they belong — the out-of-scope items in Part 3, and the per-device deferrals in the Device Configuration document — and are not relisted here. Phase 2 builds on the foundation and resolves those conditions as the hardware it introduces calls for them.

**Validation status.** The plan was written before the fabric was built and has since been partially executed. The first bench run (2026-09-11) brought all three devices up and ran every test that does not require a host on a test port or a vantage point outside the lab — roughly half the plan. Those results, the deviations found, and one test shown to be badly conceived are recorded in that run's record.

What remains unproven is the traffic behaviour: that the servers segment actually reaches the internet and is translated, that the management segment actually cannot, and that nothing initiates inbound from outside. Those tests need a host on a test port and an external vantage point. Until they run, Phase 2 should treat the fabric's filtering and NAT posture as configured-and-inspected rather than demonstrated.

### 6.9 Loose-end closures

Two build-time exceptions bypassed the fabric's own rules during construction. Both were parked for this card, because each is closed only once the fabric is proven ready to stand without it.

**6.9.1 Core-switch build lifeline (dc01 LOM2 → home router)**

*What it was.* A temporary link from dc01's second LOM straight into the core during build, bypassing segmentation — a working shortcut while the fabric was being stood up.

*Decision.* Removed. The fabric no longer needs it, and leaving it in place would be a standing hole through the very segmentation the fabric enforces.

*Status: closed.* The link is physically disconnected. No path bypasses the segmentation design.

**6.9.2 Break-glass lab-access host (PAW-01)**

*What it was.* A privileged host with a path into the lab, used during build.

*Decision.* The T470 is the sole lab-access host and carries the PAW-01 name. The break-glass terms apply to it: no standing access, physical reconnect required to use, use logged. It is not left permanently connected — reconnection is a deliberate, logged act when break-glass is invoked.

*Current state, honestly stated.* PAW-01 is at present a vanilla Ubuntu install with general internet access — not a hardened, single-purpose privileged access workstation. The control that is real today is **physical**: the host cannot reach the lab without being deliberately reconnected, and that reconnection is logged. Host hardening — dedicating the machine to lab administration, restricting general internet use and installed software — is deferred future work, not a Phase 1 claim. The document does not describe PAW-01 as a hardened PAW, because it is not one yet; it describes the physical break-glass control that genuinely holds.

The PN52 is explicitly **not** a lab-access host. It is a document and Git workstation only, sitting outside the fabric's trust boundary, and is used for console access at the bench only when physically cabled to a device's console port. Separating the two roles onto two machines is what lets the lab-access host stay disconnected by default while remote document work continues uninterrupted.

*Status: decided, applied at the bench.* PAW-01 ends Phase 1 physically disconnected from the lab, reconnected only under break-glass.

## 7. Acceptance criteria

- The validation plan covers all five fabric claims plus the standalone management-switch check — six groups, every test with a method, an expected result, and the connections it requires.
- Every test traces to an approved Phase 1 design; no test invents behaviour the designs do not specify.
- Every test is runnable at the bench as written, against the fabric as the hardware actually permits it to be built — no test asks for a configuration the platform cannot hold.
- Each test states the connections it requires, so that a test which cannot yet run is distinguishable from a test that failed.
- Results live in dated run records, not in this document; the plan stays a stable statement of what correct looks like.
- The handover states what Phase 1 delivered, what Phase 2 inherits, and an honest validation status — which claims are demonstrated and which are only configured-and-inspected.
- The two loose ends are closed, with their current status recorded.
- Deferrals and placeholders are cross-referenced to where they live, not relisted.

## 8. References

- **Physical Port Map** — the port assignments the tests reference when exercising a link.
- **Segmentation Design** — the trust boundaries the deny tests prove hold.
- **VLAN and IP Address Plan** — the addresses every test uses as source and destination.
- **Routing and ACL Design** — the routing, ACL, and NAT behaviour the expected results check against.
- **Device Configuration and Bring-Up** — the configuration under test; every test proves a line of it.
- **Network Fabric Bring-Up Run (2026-09-11)** — the first execution of this plan, and the source of the corrections in version 1.1.
- **Naming Convention** — the convention behind the hostnames used here *(not yet written)*.
- **Master Document** — where this document sits in the repository.