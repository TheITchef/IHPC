# Why This Project Uses Decimal Versioning

## The decision

Documents in this project carry a version number that increases by one tenth (0.1 → 0.2 → 0.3), sits below 1.0 until first approval, and becomes 1.0 at first approval. I examined the alternatives before settling on this, so the choice is deliberate, not accidental.

## The main alternative I considered: Semantic Versioning

**Semantic Versioning (SemVer)** is the widely-used standard for software: a version looks like `MAJOR.MINOR.PATCH` (for example `2.4.1`), where:

- **MAJOR** increases on a breaking change (something that isn't backward-compatible),
- **MINOR** increases when features are added compatibly,
- **PATCH** increases on small backward-compatible fixes.

SemVer is excellent for *software releases* — its whole purpose is to signal, to people depending on your code, whether an upgrade will break them.

## Why I chose decimal instead

SemVer solves a problem this project doesn't have. My deliverables are **documents**, not software packages with downstream dependents who need breaking-change signals. Three MAJOR.MINOR.PATCH fields would be ceremony without meaning here — there's no "breaking change to a document" in the SemVer sense.

A simple decimal number does everything I actually need:

- It **orders** revisions (0.3 comes after 0.2).
- It **signals approval status** at a glance — below 1.0 means never yet approved; 1.0 or above means approved at least once.
- It's **easy to reason about** for a single-operator documentation project.

The lesson: match the versioning scheme to the *thing being versioned*. SemVer fits software with dependents; a plain decimal fits a documentation project. Choosing the simpler scheme on purpose — after understanding the alternative — is itself the disciplined move.

## One-line takeaway

Considered SemVer, chose decimal: documents aren't software, so I version them in a way that fits documents — ordered, and clear about approval status.