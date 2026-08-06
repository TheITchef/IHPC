Learning — VLAN and IP Address Plan

Date: 2026-08-06 · Card: Phase 1, VLAN and IP Address Plan · Deliverable: docs/design/vlan-ip-plan.md

Why these choices

Tens, not hundreds (10/20/30). VLAN IDs stop at 4094; the second octet of an address stops at 255. Tens keep a clean echo (VLAN 20 → 10.20.x.x) with room for 250+ segments. Hundreds break the echo at the third segment — 10.300.x.x does not exist. A "different" scheme that fights the address math is a trap, not a flourish.

10.x, not 172.x. The ISP's internal resolver sits at a 172.x address. Staying on 10.x avoids the clash.

/30 for transit, not /31. Both fit a two-end link. /31 wastes nothing and is arguably more correct; /30 is the universally recognised choice and reads without a pause. Chose /30, recorded /31 as the considered alternative.

DHCP defined but dormant. The .100–.239 range is drawn now but serves no clients until Phase 3. Designing the empty range now keeps the address map complete and expansion-ready instead of retrofitted. Servers and management stay static — an anchor must answer at a known address, especially a controller when its server is down.

Exhaustion check

Not a risk — a /24 against single-digit host counts is a few percent used, and 10.0.0.0/8 holds ~250 possible VLANs. The real future concern is overlap, not exhaustion: when cloud is added, on-prem and cloud need non-overlapping slices of 10.x or they cannot route. A decision for the hybrid card.