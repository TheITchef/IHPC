# Segmentation Design — defines the separated areas of the lab network and the reason for each boundary.

**Status:** Approved · **Version:** 1.0 · **Last updated:** 2026-07-31 · **Owner:** Ioannis Mintzivyris

## 2. Overview

This document divides the lab network into separated areas and gives the reason for each one.

It does not assign VLAN IDs or subnets — that is Card 3. It does not decide which areas may talk to each other — that is Card 4.

Everything after this card is built per segment. Getting a boundary wrong here means re-cutting the network later.

## 3. Scope

Covers the separated areas of the lab network, what belongs in each, and why each boundary exists.

Does not cover VLAN IDs, subnets, gateways, routing, filtering rules, or device configuration.

Covers Phase 1 only. Segments needed later for virtual machine traffic, storage and vMotion are named as future work, not designed here.

## 4. Dependencies

- Physical Port Map (Approved 1.2) — what is cabled to what, and the interfaces still free.

## 5. Deliverables

A list of the segments, each with a purpose and a stated boundary.

A record of what sits outside the segmentation, and why.

A list of segments deferred to later phases.

## 6. Detailed content

### 6.1 How boundaries are decided

A segment exists when two things must be kept apart.

Two questions decide it:

- How exposed is it? How likely is something here to be attacked or infected.
- How much damage if it is lost? What an attacker gains by owning it.

Things that differ on either question do not share a segment.

A boundary is only added when it can be maintained. A separation nobody can keep working gets bypassed, and a bypassed control is worse than a documented one.

### 6.2 The segments

**Management**

Purpose: administration of the lab. Network device logins, server hardware controllers, and the workstation used to reach them.

Boundary: everything on the out-of-band switch. Nothing outside it may open a connection inward.

Reason: control of this segment means control of every device in the rack. It is the most sensitive area and the least exposed.

**Servers**

Purpose: the machines that run the lab's services.

Boundary: the server interfaces on the core switch.

Reason: internal, moderately exposed, and holds the services worth protecting. Separate from management because a compromised server must not reach hardware controllers.

**Transit**

Purpose: the link between the core switch and the edge router.

Boundary: those two interfaces only. No host sits here.

Reason: it carries traffic between the lab and the internet and nothing else. Keeping it separate means the edge router touches one lab segment instead of all of them.

This is a link, not an area with hosts. It is listed because it is a separate broadcast domain and needs its own subnet.

### 6.3 Inside the management segment

The management segment holds two kinds of thing.

**Hardware controllers** — iDRAC on the Dell servers, iLO on the HPE server. Whoever reaches one controls that machine fully, including power and boot. They are rarely patched and ship with weak defaults.

**Device logins** — the command line of the two switches and the router.

Larger sites usually split these into two segments, because controllers carry more risk. They stay together here. There are three controllers, one rack, one administrator. A second segment would add work without changing what an attacker could reach, because both would sit on the same switch and the same uplink.

Controllers are limited by rules inside the segment instead. Splitting them out stays open for Phase 4.

### 6.4 What sits outside the segmentation

**The home network and the internet.** Both sit beyond the edge router. The lab treats them as untrusted. Nothing outside may open a connection into the lab.

**The upstream distribution switch.** It is unmanaged. Anything plugged into it shares a broadcast domain with the router's WAN port. The lab has no control there, so nothing may be assumed about it.

**PAW-01 (the PN52).** A general-purpose machine. It has no connection to the management segment. If it is ever reconnected, it is a deliberate act under defined conditions.

**Console access.** Building and recovering devices is done by cable, straight to the device. This path ignores the segmentation entirely and is protected by physical access, not by rules.

### 6.5 Deferred segments

These are named so the design leaves room for them. They are not built in Phase 1.

- **Virtual machine traffic** — for machines running on esxi01 and ms01. Phase 3.
- **Storage** — Phase 3.
- **vMotion** — moving running machines between hosts. Phase 3.
- **DMZ** — needed once anything is published to the internet. Phase 4 or later.
- **Hybrid connectivity** — the link to cloud services. Later phase.

esxi01 has six free interfaces, which is what makes the first three possible without new hardware.

## 7. Acceptance criteria

- Every segment has a stated purpose and a stated boundary.
- Every boundary has a written reason.
- No VLAN ID, subnet, or filtering rule appears in this document.
- Everything outside the segmentation is listed, with the reason it is outside.
- Deferred segments are named, with the phase that builds them.

## 8. References

- Physical Port Map — what is cabled to what
- Master Document — where this document sits in the repository