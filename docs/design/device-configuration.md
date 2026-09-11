# Device Configuration and Bring-Up — turns the Phase 1 designs into the actual commands that configure each device.

**Status:** Draft · **Version:** 1.1 · **Last updated:** 2026-09-11 · **Owner:** Ioannis Mintzivyris

## 2. Overview

This document is where the Phase 1 designs become device configuration. Cards 1 through 4 recorded intent and deliberately kept commands out. This card brings those commands home.

It configures three devices, referred to throughout by role:

- **itc-uvy-core01** — the core switch
- **itc-uvy-rtr01** — the edge router
- **itc-uvy-oob01** — the management switch

Each gets its own section. The full hostname is used in section headings, configuration, and wherever a specific device must be named without ambiguity; the role name is used in the body prose. Three things that span all of them — the order devices are brought up in, the rollback target, and the reasoning behind the config choices — sit in their own sections rather than being repeated per device.

Configuration is shown as **annotated excerpts**: the load-bearing commands, each with a short note on what it does and why. It is not a paste-ready script. A reader learns what each device is configured to do and the reason for it; the operator at the console fills the routine scaffolding around these excerpts.

The document covers Phase 1 only. All network devices are administered by serial console in this phase — in-band SSH is deferred.

**Version 1.1 records what the hardware actually permits.** Version 1.0 was written before the fabric was built. The first powered bring-up found one instruction the platform does not support, and several details worth correcting. Those changes are folded into the sections below rather than listed separately, each with the reason it changed. The bring-up run record holds the evidence.

## 3. Scope

Covers the Phase 1 configuration of the three network devices — itc-uvy-core01, itc-uvy-rtr01, and itc-uvy-oob01 — as annotated command excerpts, plus the bring-up order and rollback that span them.

For each device this means: the VLANs, interfaces, routing, and filtering that the approved Phase 1 designs assign to it, and nothing beyond what those designs settled.

Does not cover:

- **The servers and their services.** esxi01, ms01, and dc01 are hosts on the network, not devices this card configures. dc01's DHCP and DNS roles belong to a later phase; here it is simply a host on the Servers segment.
- **In-band SSH to the network devices.** Phase 1 administration is by serial console. In-band management is a later expansion (see Routing and ACL Design, deferred flows).
- **Anything the approved designs deferred** — remote administration, management DNS and time, the Phase 3 server-side segments. These are recorded as deferred in their own design documents and are not reopened here.

Covers Phase 1 only.

## 4. Dependencies

Every design this card implements comes from an approved Phase 1 document. Card 5 adds no new design decisions — it applies the ones already settled.

- **Physical Port Map** (Approved 1.2) — which device interface lands on which port. Governs every access-port, trunk, and uplink assignment in the device sections.
- **Segmentation Design** (Approved 1.0) — the three segments and the trust boundaries between them. The reason the config separates traffic the way it does.
- **VLAN and IP Address Plan** (Approved 1.0) — the VLAN IDs, subnets, and gateway addresses the config applies. The SVIs, the transit addressing, and the DHCP range all come from here.
- **Routing and ACL Design** (Approved 1.0) — where routing lives, the transit link, the static return routes, and the management flow matrix. The core's inter-VLAN routing, the edge's NAT and firewall posture, and the management ACL all implement this document.

## 5. Deliverables

- **An itc-uvy-core01 configuration** — the VLAN database, the three SVIs that act as segment gateways, inter-VLAN routing, the access and trunk port assignments, and the management ACL applied to VLAN 10.
- **An itc-uvy-rtr01 configuration** — the WAN interface, NAT, the stateful firewall implementing the inbound deny, and the static return routes to the lab subnets.
- **An itc-uvy-oob01 configuration** — a layer-2 access switch presenting VLAN 10, with its uplink to itc-uvy-core01.
- **A bring-up order** — the sequence the three devices are configured and brought up in, with the dependency reason for that order.
- **A rollback target** — the state each device returns to if bring-up fails, and the order to back out in.

Each configuration is a set of annotated command excerpts, not a paste-ready script.

## 6. Detailed content

### 6.1 itc-uvy-core01 — core switch (WS-C3850-48T)

The core switch is the Layer 3 boundary for the lab. It holds the three segment gateways, routes between them, and enforces the management ACL. Its configuration is the largest in this card because it carries the most responsibility.

**Device identity**

The device is given its target hostname — the name the Physical Port Map records for it. Bring-up is where the name is applied.

```
hostname itc-uvy-core01
```

Only the hostname is set here. Users, SSH, logging, and time are all Phase 1 deferrals — serial console administration, no in-band management — so device identity in this phase is the name and nothing more. The naming convention this name follows is documented separately.

**VLAN database**

The three Phase 1 VLANs are declared before anything references them. An SVI or an access port that names a VLAN not in the database is rejected or silently inert, so the database comes first.

```
vlan 10
 name mgmt
vlan 20
 name srv
vlan 30
 name transit
```

The names match the VLAN and IP Address Plan. VLAN 1 is left unused — nothing is placed in it, per the plan's decision to keep the default VLAN empty.

**Verify the VLAN database is clean before declaring.** The VLAN database does not live in the running configuration. It is held in `vlan.dat` in flash, and it survives `write erase`. A switch can show a completely clean running configuration while still holding VLANs from an earlier life. Check with `show vlan brief` before configuring, and remove any VLAN that is not part of this design with `no vlan <id>`. A genuine factory reset needs `write erase`, `delete flash:vlan.dat`, and a reload.

**Layer 3 forwarding**

The switch is told to route. Without this, the interfaces below answer for their own subnets but traffic stops at each segment boundary — the switch behaves as three separate Layer 2 segments with addresses, not a router. This one line is what makes the core switch the lab's single routing authority.

```
ip routing
```

**Segment gateways (SVIs)**

Each VLAN gets a virtual interface that acts as its gateway — the address hosts send off-segment traffic to. All three are the .1 of their subnet, per the VLAN and IP Address Plan.

```
interface Vlan10
 ip address 10.10.0.1 255.255.255.0
 no shutdown
interface Vlan20
 ip address 10.20.0.1 255.255.255.0
 no shutdown
interface Vlan30
 ip address 10.30.0.1 255.255.255.252
 no shutdown
```

VLAN 10 and 20 are the host-segment gateways (management, servers). VLAN 30 is the core end of the transit link to the WAN router — a /30, two usable addresses, .1 here and .2 on the router.

An SVI holds its line protocol down until at least one live port exists in its VLAN. In Phase 1 that means the management and transit gateways come up once their fabric links are live, and the servers gateway stays down until a host is connected. This is expected, not an error: the gateway exists first and waits for its segment to be populated. A segment whose SVI is down also has no connected route in the routing table, for the same reason.

**Interface activation policy**

Phase 1 brings up only the interfaces the network fabric itself requires. No server or host is live in this phase; devices are introduced later, in order, each when its phase calls for it. Every interface without a settled Phase 1 purpose stays administratively shut, with a description recording what it is reserved for. An interface is activated where and when the project needs it, never speculatively.

**Active interfaces (Phase 1)**

Two ports carry the fabric and are brought up now: the uplink to the management switch, and the transit link to the router.

```
interface GigabitEthernet1/0/48
 description uplink to itc-uvy-oob01
 switchport mode trunk
 switchport trunk allowed vlan 10
 switchport nonegotiate
 no shutdown
```

The uplink to the management switch is a trunk restricted to VLAN 10 only. A trunk carries multiple tagged VLANs over one link; restricting the allowed list to VLAN 10 keeps every other segment out of the management switch. Left at its default, a trunk carries every VLAN — which would stretch the servers segment into the management switch for no reason.

`switchport nonegotiate` disables DTP, the protocol by which Cisco ports negotiate trunking between themselves. Both ends of this link are configured explicitly, so there is nothing to negotiate; leaving DTP enabled only preserves a mechanism by which some other connected device could ask to become a trunk. The design's rule for every trunk in the lab is the same: set the mode explicitly at both ends, prune the allowed VLAN list, and disable negotiation.

```
interface GigabitEthernet1/0/46
 description transit to itc-uvy-rtr01
 switchport mode access
 switchport access vlan 30
 no shutdown
```

The router-facing port is an access port in the transit VLAN. The transit link is a single point-to-point subnet, so it carries one VLAN, untagged. Placing this port in VLAN 30 is what brings the transit SVI (10.30.0.1) up and gives the core switch a Layer 3 path toward the edge.

**Deferred interfaces**

Every server-facing port stays shut in Phase 1, described by the device it awaits. The pattern each will follow when its device is introduced — access mode, placed in its segment's VLAN — is shown here for one port as the template, but no server port is brought up now.

```
interface GigabitEthernet1/0/10
 description RESERVED dc01 LOM1 activate when dc01 introduced
 switchport mode access
 switchport access vlan 20
 shutdown
```

dc01 sits on the Servers segment, so when it is introduced its port joins VLAN 20. The esxi01 and ms01 ports (Gi1/0/1–4) follow the same pattern and are held shut the same way, awaiting their devices. Which NIC on each server takes which role — and whether the two are teamed — is a host-design decision deferred to when that server is introduced; the Physical Port Map records only that both are cabled. The prepared drops (Gi1/0/23–24) likewise stay shut with no VLAN assigned until their purpose is decided. The port-to-device mapping is the Physical Port Map's; this section only records when each port goes live.

Every remaining port on the switch — those with no reserved purpose at all — is also held shut and labelled, so that no port on the device is live without a documented reason:

```
interface range GigabitEthernet1/0/5-9
 description UNUSED held shut
 shutdown
```

Descriptions are kept to plain ASCII. Punctuation such as em-dashes can be mangled by a serial console and serves no purpose in a device description.

**Management addressing block**

The VLAN plan fixes the management static range as .2–.99. For assignment, that range is sub-divided into lanes so each class of device reads by sight:

| Range | Class |
|---|---|
| .10–.19 | Admin workstations (PAW-01 at 10.10.0.10) |
| .20–.49 | Hardware controllers (iDRAC, iLO) |
| .50–.99 | Network device management addresses (deferred, in-band management) |

This sub-division extends the VLAN and IP Address Plan and is recorded here because assignment first happens at bring-up. It is to be folded back into the VLAN plan when that document is next revised (Backlog).

**PAW-01 is the lab's only privileged access workstation.** Earlier revisions of this document referred to PAW-02; that identity no longer exists. The single privileged access workstation is PAW-01, which reaches the lab either by connecting to a management-segment port or by serial console, under the break-glass terms recorded in the Validation and Handover document.

**Management ACL**

The management segment is default-deny. Only two flows are sanctioned, both originating at PAW-01 (10.10.0.10), per the flow matrix in the Routing and ACL Design.

```
ip access-list extended MGMT-IN
 permit tcp host 10.10.0.10 <controllers> eq 443
 permit icmp host 10.10.0.10 <controllers-and-devices>
 deny ip any any log
```

Two distinct jobs are happening here, and the design keeps them separate:

- **The `deny ip any any log` is what this ACL enforces.** Applied at the VLAN 10 gateway, it stops any other VLAN — the servers segment above all — from routing into management. The log records what the deny catches. This is the real, active protection of the segment.
- **The two permits document sanctioned intent, not active enforcement.** PAW-01 and the hardware controllers both sit in VLAN 10, so a PAW-to-controller packet is switched at Layer 2 on the management switch and never reaches this SVI. A switch SVI ACL is stateless and only sees traffic crossing the routed boundary; intra-VLAN traffic passes below it. The permits therefore record the only management flows the design allows — HTTPS to the controllers, ICMP for reachability — as a written statement of "this and nothing else," even though enforcement of the intra-VLAN flow itself would need a different tool (a port ACL, private VLANs, or a host firewall). That tighter enforcement is deferred.

**At bring-up, apply the deny alone.** The two permits name controller addresses that do not exist until server hardware is introduced, so they cannot be entered literally. Since they document intent rather than enforce anything at this interface, nothing is lost by deferring them: the segment's actual protection is the deny, and it is applied from the start. The permits are added when the controllers are assigned addresses from the .20–.49 lane.

Applied inbound on the gateway SVI:

```
interface Vlan10
 ip access-group MGMT-IN in
```

This is the single enforcement point for the management posture. Stateful return handling — permitting an outbound flow and its replies across a routed boundary — is not something a switch SVI ACL provides; that job lives on the edge firewall (the 891F), which is the stateful device in the design.

**One consequence worth knowing at the console.** With the deny in place, traffic sourced from the switch itself toward the management gateway arrives inbound on this SVI and is dropped. A ping from the core switch console to 10.10.0.1 therefore fails, and logs the drop. This is the ACL working, not a fault — but it means the management gateway cannot be reachability-tested from the device that hosts it.

### 6.2 itc-uvy-rtr01 — edge router (Cisco 891F)

The edge router is the lab's boundary with the internet. It does only what a WAN edge must: it translates addresses outbound, it blocks everything inbound, and it hands return traffic back to the core. It does not route between internal segments — it has no reason to see internal-to-internal traffic at all.

**Device identity**

```
hostname itc-uvy-rtr01
no service config
no ip domain lookup
```

As with the core switch, only the hostname is applied in Phase 1. Everything else — users, in-band access — is deferred.

Two additional lines are applied at bring-up. `no service config` disables a legacy behaviour in which the router attempts to load a configuration from the network at boot; it appears by default on a router with no saved configuration. `no ip domain lookup` stops the router attempting to resolve hostnames, which otherwise produces repeated translation attempts for any name supplied by the ISP, and makes a mistyped command hang while the router tries to resolve it.

**A note on the ISP's DHCP lease.** On a router with no saved configuration, the WAN port takes a lease as soon as it boots, and that lease supplies more than an address. It can set the device hostname, configure an external time server, and install routes. Those are the ISP's defaults, not this design's decisions: the hostname and time server are replaced or removed at bring-up. The default route the lease installs is genuinely needed and stays. Time synchronisation is a Phase 1 deferral and is settled by the later in-band-management work, not adopted by accident from DHCP.

**WAN interface (Gi8)**

The WAN port takes a public address by DHCP from the ISP. It faces the internet directly — there is no NAT device in front of it.

```
interface GigabitEthernet8
 description WAN to ISP
 ip address dhcp
 no shutdown
```

The address is DHCP-assigned from the ISP's range and can change. This is why the design's rule is absolute: nothing internal is ever pinned to this address. It exists only as the lab's way out.

**Transit interface (Gi7) — the inside link**

The port facing the core switch is the router's only internal-facing interface. It sits at the router end of the transit link, .2 of the /30.

The design first specified a routed port for this: convert Gi7 with `no switchport` and apply the address to the physical interface directly. The reasoning was sound — the transit link is a single point-to-point subnet with exactly two addresses on it, so putting the address on the port itself is the most direct expression of that. The alternative, placing the address on an SVI, is the more natural pattern when several LAN ports share a subnet, which is not the case here.

**The platform does not permit it.** The 891F's LAN side (Gi0–Gi7) is an integrated 8-port Layer 2 switch. Those ports are switchports in hardware, not by configuration, and there is no routed-port mode to convert them to. Attempting `no switchport` on Gi7 returns `% Incomplete command`, and querying the available completions shows only Layer 2 sub-keywords — `access`, `mode`, `trunk`, `priority`, `protected`, `voice`. There is no standalone form of the command, because there is nothing for it to switch the port into. Only the WAN port (Gi8) is a true routed interface on this platform.

This was confirmed at the bench during the first powered bring-up (see the bring-up run record, test V24).

**The transit address therefore sits on a VLAN interface.** VLAN 30 is declared on the router's internal switch, the SVI carries the transit address, and Gi7 is an access port placing the physical link into that VLAN:

```
vlan 30
 name transit
!
interface Vlan30
 description transit to itc-uvy-core01
 ip address 10.30.0.2 255.255.255.252
 no shutdown
!
interface GigabitEthernet7
 description transit to itc-uvy-core01
 switchport mode access
 switchport access vlan 30
 no shutdown
```

VLAN 30 is used on the router to match the core switch's transit VLAN. The two ends are access ports, so frames cross the link untagged and each device could map them to any local VLAN it liked — but a mismatch would be a reading hazard for no benefit.

**The result is equivalent.** The router holds 10.30.0.2/30 at its end of the point-to-point subnet, the core holds 10.30.0.1/30 at the other, and traffic passes between them. What changed is where the address lives on the router, not what the link does. The transit link was verified working in both directions at bring-up.

**What does not change: the router remains a WAN edge only.** Adding an SVI does not give the router a role in internal routing. What keeps it out of east-west traffic is its routing table, not its interface types: it holds one static route, to the servers segment, reachable only back across the transit link. It has no interface in any internal segment and no route to management at all. The unused LAN ports (Gi0–Gi6) are held shut so that nothing can be connected into the router's internal switch and land on the transit segment:

```
interface range GigabitEthernet0-6
 description UNUSED held shut
 shutdown
```

**NAT — outbound translation for servers only**

The WAN interface is marked as the NAT outside; the transit interface is the NAT inside. Only traffic from the Servers segment is translated.

```
interface GigabitEthernet8
 ip nat outside
!
interface Vlan30
 ip nat inside
```

The inside marking sits on the VLAN interface rather than the physical port, following the transit addressing above. Network address translation operates at Layer 3, so the marking belongs on the interface that holds the address — on this platform that is the SVI, not Gi7.

```
ip access-list standard NAT-SRV
 permit 10.20.0.0 0.0.0.255
```

```
ip nat inside source list NAT-SRV interface GigabitEthernet8 overload
```

The `NAT-SRV` list names only 10.20.0.0/24 — the Servers segment. Management (10.10.0.0/24) is absent by design, so it is never translated and has no route to the internet. `overload` lets the whole segment share the single public address (PAT — the router tracks sessions by port so many hosts fit behind one address). The rule points at the WAN interface, not its address, so a DHCP address change breaks nothing.

IOS adds `ip virtual-reassembly in` to each interface as NAT is configured. This reassembles fragmented packets so that NAT can read the port numbers it needs — only the first fragment of a packet carries them. It appears automatically and is not part of this design.

**Outbound access is granted by exception, not by default.** VLAN 20 is the only segment with an outbound path today because it is the only one with a stated reason for one. Management is deliberately excluded. Future segments (Phase 3 VM traffic, storage, vMotion) are added to NAT individually only where they have a justified need — storage and vMotion, for instance, are internal by nature and never get an outbound path. Each new segment is an explicit exception, never automatic.

**Stateful firewall (Zone-Based Firewall)**

The firewall enforces the load-bearing rule: nothing initiates into the lab from the internet. Because the WAN address is directly internet-reachable, this deny is real, not theoretical.

**Build the policy before zoning the interfaces.** The order matters more here than anywhere else in Phase 1. The moment an interface joins a zone, traffic crossing between zones is denied unless a policy permits it — and if no policy exists yet, everything stops. Building in the order below means the permits are in place before the deny becomes active. On a device administered in-band rather than by console, getting this backwards locks the operator out.

A class-map names the outbound traffic to track, and a policy-map says what to do with it:

```
class-map type inspect match-any LAB-OUT
 match protocol tcp
 match protocol udp
 match protocol icmp

policy-map type inspect INSIDE-TO-OUTSIDE
 class type inspect LAB-OUT
  inspect
 class class-default
  drop
```

`match-any` means a packet matching any one of the three protocols matches the class. `class-default` is built into every policy-map and catches whatever the named classes do not; its `drop` action is written out explicitly so that a reader need not know the implicit default.

Two zones are defined:

```
zone security INSIDE
zone security OUTSIDE
```

A zone-pair binds the policy to one direction only — inside to outside:

```
zone-pair security IN-OUT source INSIDE destination OUTSIDE
 service-policy type inspect INSIDE-TO-OUTSIDE
```

Only now are the interfaces placed in their zones — the transit side inside, the WAN outside. This is the step that makes the firewall live:

```
interface Vlan30
 zone-member security INSIDE
!
interface GigabitEthernet8
 zone-member security OUTSIDE
```

The inside zone member is the VLAN interface, for the same reason NAT's inside marking is: zone membership follows the Layer 3 interface, which on this platform is the SVI.

Inspection is what makes the firewall stateful: it remembers each outbound session, so the reply from the internet is allowed back automatically. This is where genuine established-session handling lives in the design — on the edge, not on the switch SVI, which is stateless.

The inbound deny is enforced by absence: there is deliberately no outside-to-inside zone-pair, so internet-initiated traffic has no policy permitting it and hits the ZBF default deny. Nothing inbound is written as a rule; it is simply never permitted. The two mechanisms are worth holding apart — the outbound `class-default` drop is written explicitly for readability, while the inbound deny cannot be written at all and exists only as a deliberate omission.

IOS also defines a built-in `self` zone, representing traffic to and from the router itself rather than through it. Traffic involving `self` is permitted by default unless a zone-pair is created for it, which is why console access and the router's own DHCP client are unaffected by zoning. No `self` zone-pair is created in Phase 1.

**Static return route to the lab**

The edge does not participate in internal routing, so it must be told how to return traffic to the lab. One static route points the Servers segment back at the core.

```
ip route 10.20.0.0 255.255.255.0 10.30.0.1
```

The route reads: "to reach 10.20.0.0/24, send it to 10.30.0.1" — the core's transit address. The core holds all internal routing and fans out from there.

Only VLAN 20 gets a return route — the only segment with an outbound path, so the only one that ever receives return traffic. Management (VLAN 10) is absent by design: the edge has no route to it at all, reinforcing Tier 0 isolation. If remote administration into the lab is designed later (the deferred flow — VPN or Bastion), that card decides what path management gets, if any. The omission here is deliberate, not forgotten.

Alongside this route, the routing table also carries the default route the ISP's DHCP lease installs, and may carry host routes belonging to the ISP's own provisioning network. Those are not this design's and cannot be prevented while the WAN uses DHCP; they are expected and left alone.

### 6.3 itc-uvy-oob01 — management switch (WS-C3560CG-8PC)

The management switch presents the management segment (VLAN 10) to the hardware controllers and the privileged access workstation. It does no routing and holds no IP address of its own in Phase 1: it is a Layer 2 access switch whose entire job is to place the right ports in VLAN 10 and carry that VLAN up to the core switch, where it is routed. It is capable of Layer 3 — the hardware supports routing and SVIs — but runs Layer 2 only by design, so that all routing and filtering stay on the core switch.

The device has ten ports: eight access ports (Gi0/1–8) and two uplinks (Gi0/9–10).

**Device identity**

```
hostname itc-uvy-oob01
```

Hostname only, as with the other devices. No management IP address is set: the switch's own SVI is deferred to the in-band-management card, together with SSH and the flow-matrix permit that would make in-band administration safe. In Phase 1 it is administered by console, so it needs no address to be reachable.

**VLAN database**

Only VLAN 10 is declared. The management switch carries one segment and no other.

```
vlan 10
 name mgmt
```

As with the core switch, check `show vlan brief` before declaring. The VLAN database survives a configuration erase, and any VLAN not part of this design is removed first.

**Access ports — controllers and PAW-01**

The controller ports and the PAW-01 port belong in VLAN 10, access mode. As with the core switch, only ports with a settled Phase 1 device are activated; the rest stay shut. In Phase 1 no device is live, so every access port is held shut.

```
interface GigabitEthernet0/8
 description RESERVED PAW-01 activate on break-glass connect
 switchport mode access
 switchport access vlan 10
 shutdown
```

The controller ports (Gi0/1–3, the iDRAC and iLO interfaces) follow the same pattern — access mode, VLAN 10 — and are held shut until their servers are introduced. PAW-01's port is shown as the template; it too stays shut until PAW-01 is connected under break-glass. No port is brought up while its device is off. The remaining access ports and the unused second uplink are shut and labelled as unused.

**Trunk uplink to the core switch**

The uplink carries VLAN 10 to the core switch, where the segment is routed. This is the one port that must be live for the management segment to have a gateway at all.

```
interface GigabitEthernet0/9
 description uplink to itc-uvy-core01
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 10
 switchport nonegotiate
 no shutdown
```

The trunk allows only VLAN 10 — the sole segment this switch carries. This is the matching end of the trunk configured on the core switch's Gi1/0/48; both ends restrict the allowed list to VLAN 10, so nothing else can traverse the link.

`switchport trunk encapsulation dot1q` must be applied before the mode command on this platform; the 3560 supports more than one trunking encapsulation and rejects the mode change until one is chosen. The core switch does not need the line — IOS-XE supports only 802.1Q and sets it implicitly.

**Trunking is set explicitly at both ends, and negotiation is disabled.** This port's factory default is `dynamic auto`, meaning it does not ask to trunk but will agree if something else asks. At bring-up, the core switch's explicitly configured trunk asked, and this port became a trunk on its own — before any configuration had been applied to it. That is the behaviour the design rules out. A port that will trunk on request is a port any connected device can turn into a trunk, gaining access to every VLAN the link carries. Worse, the running configuration shows nothing while the port operates as a trunk, so the documented state and the actual state disagree. Explicit mode plus `switchport nonegotiate` at both ends removes both problems.

### 6.4 Bring-up order

Configuration and power-up follow a fixed order, so that each device's dependencies exist before the next device needs them. All configuration is done by serial console, one device at a time. Nothing is brought up in parallel.

The order is driven by one rule: **a gateway must exist before the things that depend on it.** The core switch holds every gateway in the lab, so it is configured first; everything else depends on it.

**0. Verify the baseline first**

Before any device is configured, confirm what state it is actually in. Check the running configuration, the startup configuration, and — on the switches — the VLAN database, which is held separately and survives an erase. A device that is not at a clean baseline is returned to one before configuration begins, per the rollback target below. Assume nothing from the prompt: a default hostname does not prove a default device.

**1. itc-uvy-core01 — the core switch**

Configured and verified first, because it holds the routing for the entire lab. Until its SVIs exist, no segment has a gateway and no traffic can cross a VLAN boundary. Within the core switch, the order is: hostname, VLAN database, `ip routing`, the three SVIs, then the fabric ports (the uplink trunk and the transit port), then the management ACL, then the interface activation policy across the remaining ports. Once the core switch is up, the lab has its routing spine.

**2. itc-uvy-rtr01 — the edge router**

Configured second. It depends on the core switch in one direction — its static return route points at the core's transit address (10.30.0.1), which must exist first — and the transit SVI on the core must be up for the link to pass traffic. The edge router brings the lab's path to the internet online, but that path is only useful once the core is routing beneath it.

Within the router, the order is: identity, interfaces (WAN and transit), NAT, firewall, static route. The firewall comes late because it is the piece most likely to need iteration, and within it the policy is built before the interfaces are zoned. The router's WAN link becomes live as soon as the device boots, so the interval between first boot and the firewall being in place is worth keeping short.

**3. itc-uvy-oob01 — the management switch**

Configured last. It is a Layer 2 access switch whose uplink trunk depends on the core switch's VLAN 10 SVI to give the management segment a gateway. Configuring it before the core switch would leave its one live port pointing at a gateway that does not yet exist. It is the simplest device and depends on the most, so it comes last.

**On "power-up" in Phase 1**

In Phase 1 the order is a *configuration* order, not a live-traffic bring-up: no host is connected, so the only live ports are the fabric links between the three network devices. The order still matters — it is the sequence in which the devices are configured and verified — but the lab carries no host traffic at the end of Phase 1. It is a routed, filtered, empty fabric, ready for hosts to be introduced in later phases.

### 6.5 Rollback

The rollback target for all Phase 1 work is **factory default**. Phase 1 is greenfield: there is no earlier configuration to preserve, so rollback is not a restore to a previous version but a return to a clean baseline.

**What rollback means**

If a device's configuration fails verification, or a change produces an unexpected result that cannot be quickly corrected, the device is returned to factory default and its configuration is reapplied from this document. Erase the startup configuration and reload; the device comes back to baseline with no residual state. Because the configuration is documented here rather than held only on the device, nothing is lost by erasing — the document is the source, the device is disposable.

**On switches, erasing the configuration is not enough.** The VLAN database lives in `vlan.dat` in flash and survives `write erase`. A full return to factory default on a switch is `write erase`, `delete flash:vlan.dat`, then reload. Skipping the second step leaves VLANs behind that no configuration file will show.

**Granularity — per device**

Rollback is per device. A device that verifies correctly is left alone; only the device that failed is reset and retried. There is no reason to tear down a correctly-configured core switch because the management switch's configuration went wrong. Each device is configured and verified in isolation over console, so each can be rolled back in isolation.

**The core switch exception**

One dependency breaks the per-device rule: the core switch holds every gateway in the lab. Rolling back the core switch removes the gateways the other two devices depend on, so a core rollback effectively invalidates the edge router and the management switch as well — not because they were reset, but because the routing beneath them is gone. If the core switch must be rolled back after the others are up, the others are re-verified once the core is reconfigured, in the original bring-up order.

**Back-out order**

Where more than one device must be rolled back, back out in the reverse of the bring-up order — management switch first, then edge router, then core switch last. This unwinds dependencies safely: the most-dependent device is removed first, the most-depended-upon device last, so no device is ever left pointing at a gateway that has just been erased.

## 7. Acceptance criteria

- Each of the three network devices has a configuration section written as annotated command excerpts, covering the design assigned to it by the approved Phase 1 documents.
- Every VLAN, SVI, interface, routing statement, and filtering rule in the configuration traces to an approved Phase 1 design; no new design decision is introduced here.
- Routing and inter-VLAN forwarding are configured on the core switch only. The edge router performs NAT and stateful filtering and holds no internal routing beyond the static return route.
- The management ACL applies the flow matrix: default-deny into the management segment, with the two sanctioned PAW-01 flows recorded. The enforcing deny and the documented-intent permits are distinguished, and the deny is applied at bring-up whether or not the permits can yet be entered.
- Every trunk is configured explicitly at both ends, with a pruned allowed-VLAN list and negotiation disabled.
- Only interfaces with a settled Phase 1 purpose are brought up; all others are held shut with a description of what they await.
- The bring-up order is stated, with the dependency reason for each position, and includes baseline verification before any device is configured.
- The rollback target is stated as factory default, with the back-out order, the core-switch cascade, and the VLAN database recorded.
- Where the hardware does not permit what the design specified, the document records the original intent, the constraint, and the implementation actually used.
- No device is live at the end of Phase 1; the fabric is routed, filtered, and empty, ready for hosts in later phases.

## 8. References

- **Physical Port Map** — device-to-interface mapping; the source for every port assignment.
- **Segmentation Design** — the segments and trust boundaries the configuration separates.
- **VLAN and IP Address Plan** — the VLAN IDs, subnets, and gateway addresses the configuration applies.
- **Routing and ACL Design** — the routing responsibilities, edge posture, and management flow matrix the configuration implements.
- **Validation and Handover** — the validation plan for this configuration, and the break-glass terms governing PAW-01.
- **Network Fabric Bring-Up Run (2026-09-11)** — the bench record that produced the corrections in version 1.1.
- **Naming Convention** — the convention behind the device hostnames applied here *(not yet written)*.
- **Master Document** — where this document sits in the repository.