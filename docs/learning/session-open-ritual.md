# The Session-Open Ritual — how I start every session

## What the ritual is

Every work session starts with the same three steps, in order, before any new work:

1. **State the current date and time.** I say it explicitly; my collaborator never assumes the date has advanced or stayed the same since last time.
2. **Fresh-eyes review** of the previous session's deliverable — a genuine re-read, at the start of a *new* session, not same-session self-review.
3. **`git pull`** from GitHub — sync local with the remote before touching anything.

## Why each step matters

**Timestamping (step 1).** Sessions happen across different days and gaps. Anchoring each one to a real date/time keeps the project's history honest and stops wrong assumptions about how much time has passed — which matters for things like "last updated" metadata and knowing whether the remote might have moved.

**Fresh eyes (step 2).** Reviewing a deliverable at the *start of the next session* — rather than right after writing it — gives genuine distance. Same-session review sees what you meant to write; next-session review sees what's actually on the page.

**Pull first (step 3).** `git pull` brings down anything on GitHub I don't have locally. The professional habit is to pull *first, before touching anything* — so I always build on the current state, not a stale copy.

## The insight

I corrected my own mental model on the pull. It isn't tied to "just before I push" or "when I branch" — it's simply the first thing you do, every session, full stop. Push syncs *up*; pull syncs *down*; branching is a purely local operation that touches neither.

As a single operator on one machine, the pull almost always returns "Already up to date" — it does nothing visible, and it's tempting to skip. But the point is to build the reflex now, while the stakes are zero, so it's automatic on a team where it genuinely prevents conflicts and messy merges.

## One-line takeaway

Start every session the same way: state the date, re-read yesterday's work with fresh eyes, pull before touching anything.