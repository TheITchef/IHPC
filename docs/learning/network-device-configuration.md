# Learning: Device Configuration and Bring-Up

Companion notes to the Device Configuration and Bring-Up document. The design doc records what was configured; this one carries why, with the reasoning and the tradeoffs behind each call.

## NAT looks like security, but it's really just addressing

It's tempting to think NAT protects you. Outsiders can't see your internal addresses, so surely they can't reach them. But that safety is a side effect, not the job NAT was built for — NAT exists to let many hosts share few public addresses. What actually stops unsolicited traffic getting in is that there's no translation waiting for it, and a stateful firewall does that far better, without touching a single address.

So the lab uses NAT in exactly one place: at the edge, outbound, because one public address has to serve a whole segment. Between internal VLANs it uses none. Translating traffic from one internal segment to another would buy no security an attacker on the inside couldn't simply walk around — and it would cost the thing a clean address plan gives you for free, where every address announces its own segment on sight. The rule worth keeping: when you want security, reach for a firewall or an ACL. Reach for NAT only when you have an addressing problem. The one honest exception is two networks with overlapping ranges — and a plan built with care never has that.

## A switch ACL doesn't remember anything

Here's a question that catches people. You permit a request into a segment — how does the reply get back out? On a real firewall the answer is easy: it remembers the session and lets the reply through. A switch ACL on a routed interface remembers nothing. Every packet is judged on its own, with no idea a conversation is happening.

Two things make that fine here. First, the management flows that seem to need it don't cross the ACL at all — the workstation and the controllers share a VLAN, so their traffic is switched below the gateway and never meets the filter. Second, when you genuinely do need replies handled statefully — an outbound flow and its return traffic across a boundary — that job belongs on the edge firewall, which is built to track sessions. The switch filters; the firewall remembers. Knowing which box does which is what keeps a flow from being written in the wrong place.

## The management switch could route — and deliberately doesn't

The little management switch is perfectly capable of Layer 3. It can route, it can hold gateway addresses, the hardware supports it. The lab runs it as a plain Layer 2 access switch anyway, and that's a decision worth stating out loud rather than hiding.

Keeping all the routing and filtering on the core means one place makes those decisions, and the most sensitive switch in the rack carries the least configuration and the smallest surface to attack. A device that *can* do more, kept simple on purpose, is a stronger design than one that's simple because it has no choice. Capability and role are different questions, and the gap between them is where a lot of good security decisions live.

## Nothing gets switched on before it's needed

At the end of this phase, almost every port on the network is shut. Only the links between the network devices themselves are live; every port waiting for a server sits administratively down, labelled with what it's for. That can look unfinished. It isn't — it's the point.

A port with no settled purpose doesn't get configured "for now," because configuring ahead of a decision is really just guessing at that decision. The same habit scales outward: a new segment gets a path to the internet only when there's a stated reason for one, added as a named exception — never handed out by default. Storage traffic and machine-migration traffic, when they arrive, get no outbound path ever. The cost is a little more work each time something is introduced. What you buy is never having to wonder why something was reachable before anything needed it to be.

## The little edge router is secretly two devices

The 891F doesn't behave like a plain router, and assuming it does will trip you up at the console. Its eight LAN ports are a built-in switch — Layer 2, and a switch port can't just take an IP address. Its WAN port is a separate routed interface that can. So the port facing the lab has to be told to stop being a switch port before it will hold an address, or the address goes on a virtual interface instead.

None of that is a surprise once you picture the box correctly: a small switch bolted to a router, sharing one chassis. The wider lesson is to know the shape of the hardware you're holding, not just its port count — because the shape is what decides which commands even apply.