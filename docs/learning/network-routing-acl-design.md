# Learning: Routing and ACL Design

Companion notes to the Routing and ACL Design document. The design doc records what was decided; this one carries why, with the reasoning and the tradeoffs behind each call.

## The management segment is built upside-down, on purpose

Most network segments are friendly on the way in and cautious on the way out. You let traffic reach the servers, and you watch what the servers send outward. Management is the opposite. It is the segment that controls everything else — the device logins, the hardware controllers — so it gets treated as the most trusted tier, the one everything else answers to. The rule for a tier like that is simple: control flows down from it, never up into it.

In practice that means default-deny in *both* directions. Nothing reaches a management interface, and management starts no conversations, unless a specific flow is written down with a reason. And access comes from exactly one place — a single privileged workstation that sits inside the segment as its only trusted origin. An administrator doesn't reach a controller from wherever they happen to be; they work from that one workstation.

The point isn't to filter management traffic. It's to make sure management has almost nothing reachable at all. If something elsewhere in the estate is compromised, it finds nothing to talk to.

## The edge deny is load-bearing, not decoration

On a normal home connection, the ISP hides everything behind its own address translation. You could leave a firewall wide open and still be invisible, because nothing outside can reach you directly. That safety is invisible and easy to take for granted.

This lab doesn't have it. The edge router gets a real public address straight from the ISP, and nothing sits in front of it — the lab and the home network are peers on the same distribution switch, neither behind the other. The edge is reachable from the whole internet.

That single fact changes the weight of every decision at the edge. "Deny all inbound" stops being a tidy default and becomes the thing actually holding the line. Whatever you permit inbound is exposed to the open internet, not to a friendly home LAN. So the edge denies everything inbound and allows back only the return traffic for connections the lab itself started — and nothing internal is ever tied to that public address, which is handed out by DHCP and can change anyway.

## Serial beats SSH for now, because serial isn't on the network at all

The obvious way to administer a switch is to SSH into it. The quieter, safer way in this phase is a serial console cable into the device's console port — because that path doesn't cross the network. No VLAN, no ACL, no routing decision is involved; it's a physical wire to the box.

That's why in-band SSH is deferred rather than enabled. Every service you turn on is a surface someone can reach. Administering by serial keeps the network devices' management surface at zero for now. SSH is a later expansion — a real convenience once the lab is on the bench full-time — but it's a deliberate *addition*, made on purpose, not something switched on by default because it's normal.

## Even ping gets scoped, because "just ping" is also reconnaissance

Ping feels harmless — it only asks "are you there?" But in a locked-down segment, a host that answers ping is a host that confirms it exists. If everything answers, anything that gets a foothold can map the whole segment in seconds.

So ICMP is permitted only from the one trusted workstation, to the devices it actually manages. The important restriction isn't which ports — it's the *origin*. One source may ping; nothing else may. That keeps the operational convenience (a fast up/down check when something won't respond) without handing out a free map of the segment to anything that lands inside it.

## The firewall is honest about what it is

The edge firewall today is the branch router's own built-in stateful firewall. It's stateful — it remembers the connections the lab started, so their replies come back automatically while unsolicited inbound traffic is dropped. That's genuinely enough for the posture this phase needs.

But it's a branch router doing firewall duty, not a dedicated security appliance, and the document says so plainly. A dedicated firewall is the acknowledged next step. Naming a limitation out loud is worth more than hiding it — it shows the boundary was understood, and it turns "why is your firewall just a router?" from a gotcha into a planned improvement you can already talk about.