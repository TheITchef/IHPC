# SOP Types & What "Infrastructure" Really Means

## The insight: not all SOPs belong to Operations

An SOP (Standard Operating Procedure) is a written procedure for doing something consistently. My instinct was to think of SOPs as *operational* — procedures for running and maintaining systems that already exist. But writing the Change-Management SOP made a distinction clear:

- **Governance SOPs** define how the *project itself* is conducted — how changes are made, how documents are written, how work is tracked. These are needed in **Phase 0**, at the very start, *before* any infrastructure exists — because they're the rules everything else follows.
- **Operational SOPs** define how the *built infrastructure* is run and recovered — deployment steps, rollback procedures, disaster recovery. These are written **when the infrastructure they govern actually exists**, not before.

The lesson: an SOP written in Phase 0 (when there's nothing to operate yet) isn't a contradiction — it's a *governance* SOP, and it's meant to exist before the building starts. Writing the rules first, then building, is deliberate.

## The related clarification: "infrastructure" is broader than the obvious

While scoping the project I also clarified what "infrastructure" actually covers. The instinct is to picture the physical trio — compute, storage, memory (the hardware). But infrastructure scope is much broader: it spans **identity, networking, security, monitoring, and recovery** as well — with Active Directory (identity) as a particular focus area for this project.

The lesson: "infrastructure engineering" isn't just racking servers. The identity layer, the network design, the security model, and the operational discipline are all part of the same scope — and often the harder, more senior part.

## One-line takeaway

Governance SOPs come first (Phase 0, before anything is built); operational SOPs come later (when there's something to operate). And "infrastructure" spans identity, network, and security — not just the hardware.