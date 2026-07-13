# Change-Management SOP — defines how changes to project documents and work are proposed, tracked, and reversed.

**Status:** Approved · **Version:** 1.0 · **Last updated:** 2026-07-13 · **Owner:** Ioannis Mintzivyris

## 2. Overview

This document defines how changes are made within the project: how a piece of work moves from idea to finished, how that movement is recorded in each document's metadata, and how a change is undone if it fails review. It covers the project's own conduct — documents, tracking, and version history — not the future infrastructure the project will build. Anyone following this SOP should be able to make a change safely, leave a clear trail behind it, and recover cleanly when something needs to go back.

## 3. Scope

Covers changes to the project's documents and tracked work: how work moves through its lifecycle, how each document's metadata records that movement, and how a change is proposed, committed, and reversed. Does not cover changes to the infrastructure the project builds or the systems that run on it — these are governed by their own change-control procedures once that infrastructure exists (see Master Document for the phase breakdown, and later phases for the procedures themselves). Does not cover document format or the metadata block's contents (see Documentation Standard) or where documents live in the repository (see Master Document).

## 4. Dependencies

Requires the Documentation Standard — this SOP governs changes to the metadata block whose fields (Status, Version, Last updated, Owner) are defined there.

## 5. Deliverables

- A defined status vocabulary — the complete set of values a document's Status field may hold.
- Metadata update rules — the conditions under which Status, Version, Last updated, and Owner change.
- A change procedure — the steps for making, recording, and reversing a change, including the failed-review return loop.

## 6. Detailed content

### 6.1 Status vocabulary

A document's Status field holds exactly one of three values. No other values are used.

| Status | Meaning |
|---|---|
| **Draft** | The document is being written or revised for the first time. It has not yet passed review. |
| **In Revision** | The document was reviewed, did not meet its acceptance criteria, and has been returned for rework. It differs from Draft in one way that matters: it has already failed a review at least once. |
| **Approved** | The document has passed review against its acceptance criteria and is considered complete. Later changes return it to Draft or In Revision. |

The distinction between *Draft* and *In Revision* is deliberate. A cold reader scanning the project can tell, from the status alone, which documents are new work and which are recovering from a failed review — the project's real history stays visible rather than hidden behind a single "draft" label.

### 6.2 Document lifecycle

Every document moves through the same lifecycle, tracked as it goes. Two things move in parallel: the document's position on the board (where the work is) and its Status (what has been decided about it). They are kept in step.

A document begins as an item on the board before any writing starts. Once work begins, the document is written — its Status is **Draft** — and when the author considers it complete, it is submitted for review. Review checks the document against its own acceptance criteria.

Review has two possible outcomes:

- **Pass.** The document meets its acceptance criteria. It moves to Done and its Status becomes **Approved**.
- **Fail.** The document does not meet its acceptance criteria. It returns to the work stage for rework, and its Status becomes **In Revision**. It is then rewritten and submitted for review again. This loop repeats until the document passes.

A document under review keeps the Status it arrived with — **Draft** the first time, **In Revision** on any later pass. Status records the *outcome* of review, not the fact that a review is happening; the board already shows that a document is under review. Status changes only when a review concludes: to Approved on a pass, to In Revision on a fail.

Once Approved, any further change returns the document to the work stage and its Status to **Draft** or **In Revision**, and it travels the lifecycle again. No document is ever edited in place while Approved — a change always re-enters the lifecycle.

### 6.3 Metadata update rules

The metadata block defined in the Documentation Standard carries four fields. This section defines when each one changes. It does not restate what the fields mean — that is defined in the Documentation Standard.

**Last updated** changes on every edit. Any change sets this field to the date the change was made. It never carries a stale date.

**Status** changes only when a review concludes or a document is reopened, as defined in section 6.2: to Approved when a review passes, to In Revision when a review fails, and to Draft when an Approved document is reopened for a new change.

**Version** is a single decimal number that increases by one tenth (for example 0.3 to 0.4, or 1.2 to 1.3) each time a document re-enters the lifecycle for a substantive change. Three rules govern it:

- The number increases at the **start** of a change — the same moment Status returns to Draft — not when the change is approved. A document being reworked therefore carries its new number while still in Draft, so no two versions ever share a number.
- A document sits below **1.0** until it is first Approved, at which point it becomes **1.0**. A version below 1.0 means the document has never been approved; 1.0 or above means it has been approved at least once.
- A change is **negligible** — updating Last updated only, with no version increase — if it does not change what the document says, only how it reads: corrections to spelling, grammar, punctuation, or formatting, and fixes to plainly mistyped names, dates, or broken links. Any change to a rule, definition, value, decision, scope, or acceptance criterion — or any addition or removal of meaningful content — increases the version. Where there is doubt, the version increases.

**Owner** changes only when responsibility for the document formally moves to another person.

### 6.4 The change procedure

This section describes the steps of making a change. It gives the order of the steps and what each achieves; the mechanics of the version-control commands are defined in the Master Document and are not repeated here.

**All document and task work happens on a feature branch. The `main` branch is only ever edited for board updates — never for document or task work.**

A change follows the same path every time:

1. **Open the work.** The change starts on its own branch, one branch per change, and the document's metadata is updated at this point: Status returns to Draft, the version increases, and Last updated is set to today. The document's board card moves into the work stage.

2. **Do the work.** The document is edited on the branch. Progress is committed as the work proceeds, each commit following the project's commit convention.

3. **Submit for review.** When the author considers the change complete, it is opened for review — the change is proposed for merging, with a description of what changed and how to check it. The board card moves to review.

4. **Review.** The change is checked against the document's acceptance criteria, with the two outcomes defined in section 6.2. On a pass, the change is merged and the document's Status becomes Approved. On a fail, the change returns to the work stage, its Status becomes In Revision, and the loop repeats from step 2.

5. **Close the work.** Once merged, the branch is removed and the board card moves to Done. The change is complete and recorded in the project's history.

A negligible change, as defined in section 6.3, follows the same path but updates Last updated only, without a version increase.

## 7. Acceptance criteria

- The status vocabulary is a closed set of three values (Draft, In Revision, Approved), and no document uses a status outside it.
- The document lifecycle is defined, including the failed-review return loop and the parallel movement of board position and Status.
- Each metadata field (Status, Version, Last updated, Owner) has a stated rule for when it changes.
- The version-numbering rule defines when the number increases, when it does not, and what "negligible" means.
- The change procedure gives the ordered steps of making a change and points to the Master Document for version-control mechanics rather than restating them.
- This document itself complies with the Documentation Standard: it carries the metadata block, follows the eight-part format, and obeys the writing rules.

## 8. References

- Documentation Standard — the eight-part format, the metadata block's fields, and the writing rules this document follows.
- Master Document — the version-control workflow and the board columns this SOP's procedure runs on.
- Project Charter — the project's purpose and canonical audience.