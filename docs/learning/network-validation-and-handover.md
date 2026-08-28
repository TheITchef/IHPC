# Network Validation and Handover — the reasoning

*Learning companion to `docs/design/validation-and-handover.md`. The thinking behind Card 6, not the spec. Primary source for the phase blog write-up.*

Card 6 closed Phase 1. The fabric was already built — three network devices configured into a routed, filtered network. Card 6's job: prove it works, and record what the phase leaves behind.

One constraint shaped everything: the lab was powered off and I worked remotely. I couldn't run a single test. So the card became writing validation you can't yet execute, for a fabric with no hosts — and being honest about the gap between "built" and "proven."

Five decisions worth keeping.

## 1. A plan, not results

Obvious move: wait until the bench, write plan and results together.

I split them. The plan states every test, its command, and the expected result now; the actual column fills at the bench.

Tradeoff: tests written blind may not survive contact with hardware. Waiting avoids that.

Design-now won because writing the expected result *is* design work — deciding what "correct" means forces a re-read of the design while it's fresh, and it caught real gaps (an SVI that won't come up without a host, a routed-port step easy to forget). A plan is a checklist you execute, not something you improvise tired at a console.

## 2. Validating an empty fabric

Phase 1 ends with no hosts. So "server A pings workstation B" isn't testable — neither exists.

Two options: skip host-level tests until hosts arrive, or find stand-ins.

I used stand-ins, stated explicitly. Config-state tests run from the device console. Traffic tests use a technician's laptop on a test port as the source. Where a test needs something that doesn't exist yet, the method names the stand-in.

The point: an empty fabric is still testable — you validate the *plane* (routing works, denies hold, NAT scopes) even with nothing on it. What you can't yet test (real host-to-host flows) is named as such, not faked.

## 3. Tests that prove more than one thing

V23 — "management can't reach the internet" — fails for three independent reasons: no NAT entry, no return route, and the management ACL all block it.

Tempting to present it as a clean NAT test. It isn't.

I recorded it as over-determined, and pointed to the two tests (V20, V22) that isolate the NAT half. The end-to-end test still earns its place — it proves the result that matters (management is isolated) — but a plan that pretends each test isolates one variable is lying about what a pass means.

## 4. Record as-found, fix as its own change

A test fails at the bench. The reflex is to fix the config and update the expected result so the plan looks clean.

Don't. The plan records what the fabric did on first bring-up. A mismatch is written as-found; the fix is a separate change with its own trail.

A validation plan quietly edited to match reality isn't a record, it's a story. The value is in the honest first-run log.

## 5. Not overclaiming the PAW

The break-glass host (PAW-01, the T470) is a vanilla Ubuntu laptop with internet access — not a hardened, single-purpose privileged access workstation.

Easy to write it up as a "PAW" and move on. The design language even invites it.

I described it as it is. The control that's real today is physical: the host can't reach the lab without a deliberate, logged reconnect. Host hardening is deferred, named as future work — not claimed as done.

This is the one that matters most in a portfolio. A homelab dressed up as more than it is fools no one who reads closely. Stating the honest boundary — real physical control, hardening still to do — is more credible than a title the machine hasn't earned.