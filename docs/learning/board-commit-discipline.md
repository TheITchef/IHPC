# Board-Commit Discipline — where board moves live

## The rule

Kanban board changes and task work never mix in the same commit.

- **Board moves** (a card changing column, a card being added) are committed straight to `main`, on their own, with a commit message prefixed `Board:`.
- **Task/document work** happens on a feature branch, never directly on `main`.

The two are kept strictly separate.

## Why

The board and the work it tracks are two different kinds of thing. The board records *where the work stands*; the branch holds *the work itself*. Bundling them into one commit blurs that line — a reviewer reading history couldn't tell "this commit moved a card" from "this commit changed a document."

Keeping board moves as their own `Board:` commits on `main` means:

- The board is always current on `main`, visible to anyone at a glance, without digging into branches.
- Feature branches stay purely about their deliverable — no stray board edits riding along.
- History reads honestly: `Board:` commits are bookkeeping; `Task:` commits are deliverables.

## The related timing rule

The board should *lead* the work, not trail it. When a task starts, its card moves into WIP (a `Board:` commit on `main`) **before** the feature branch is cut — so the board reflects reality as work begins, not after it's done.

## One-line takeaway

Board moves go on `main` as their own `Board:` commits; task work goes on a feature branch. Never mix the two — and move the card *before* you start the work.