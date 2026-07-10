# Documentation Standard — defines how every document in this project is written.

**Status:** Draft · **Version:** 0.1 · **Last updated:** 2026-07-10 · **Owner:** Ioannis Mintzivyris

## 2. Overview

This document defines the format and writing rules for every document in the project. It describes the parts every document must contain, the metadata block every document carries, and the writing rules that apply throughout. A person who has never seen this project should be able to write a compliant document using this Standard alone.

## 3. Scope

Covers document format, structure, and writing rules. Does not cover when and how document metadata is updated (see Change-Management SOP) or where documents live in the repository (see Master Document).

## 4. Dependencies

Requires the Project Charter — the project's purpose and audience are stated there and are not restated here.

## 5. Deliverables

This document itself: a single standard that all project documents follow.

## 6. Detailed content

### 6.1 Writing rules

These rules apply to every part of every document.

**Rule 1 — Point, don't copy.** Every fact has exactly one home. If information already lives in another document, link to it instead of restating it.
*Example: a design document needing the project's audience links to the Charter — it does not repeat the audience description.*

**Rule 2 — Explain jargon on first use.** Give the proper term plus a plain-language meaning the first time it appears. After that, use the term alone.
*Example: "The switch uses VLANs (virtual networks that separate traffic on shared hardware). Each VLAN is assigned..."*

**Rule 3 — Describe results, not steps.** Frame work by the outcome achieved, not the actions taken.
*Example: write "Core switch segmented into three VLANs, isolating management traffic" — not "Logged into the switch, created VLAN 10, then VLAN 20..."*

**Rule 4 — Write for a mixed audience.** Readers range from recruiters to senior engineers. Plain English is the default; technical depth is welcome, but it must never be the only way to understand a sentence.
*Example: "Replication (how domain controllers keep their copies of the database in sync) is monitored daily."*

**Rule 5 — Keep it as short as the content allows.** Every sentence must earn its place. If a section says nothing, it says "None." in one line rather than padding.
*Example: a Dependencies section with no dependencies reads "None." — not a paragraph explaining why there are no dependencies.*

### 6.2 The metadata block

Every document begins with a metadata block, placed directly under the title and above the eight parts. It is a header, not a ninth part.

| Field | Meaning |
|---|---|
| Status | Where the document stands (e.g. Draft, Approved) |
| Version | The document's version number |
| Last updated | Date of the most recent change |
| Owner | Who is responsible for the document |

*Example:*
> **Status:** Draft · **Version:** 0.1 · **Last updated:** 2026-07-10 · **Owner:** Ioannis Mintzivyris

The rules for when and how these fields change are defined in the Change-Management SOP. This document defines only what the block contains and where it sits.

### 6.3 The eight parts

Every document contains the following eight parts, in this order. The running example is a fictional VLAN Design document.

**1. Title.** One H1 line naming the document, followed by a one-line purpose statement.
*Belongs:* the name and one sentence on why the document exists. *Stays out:* everything else.
*Example: "# VLAN Design — defines the network segments of the lab and why they exist."*

**2. Overview.** A short plain-English summary a non-technical reader can follow.
*Belongs:* what this document covers, in a few sentences. *Stays out:* technical detail, justifications.
*Example: "This document describes how the lab network is divided into separate segments to keep management, server, and client traffic apart."*

**3. Scope.** What the document governs — and explicitly what it does not.
*Belongs:* boundaries, stated in both directions. *Stays out:* content that belongs to the excluded areas.
*Example: "Covers VLAN numbering and purpose. Does not cover IP addressing (see IP Plan) or switch configuration (see procedures)."*

**4. Dependencies.** What must exist or be true before this document applies.
*Belongs:* genuine preconditions, which deserve naming even at the cost of some noise. "None." if empty. *Stays out:* mere related reading — that belongs in References.
*Example: "Requires the Security Zone Model — VLANs map onto zones defined there."*

**5. Deliverables.** The concrete outputs this document produces or defines.
*Belongs:* tangible, nameable outputs. *Stays out:* activities and effort — outputs only.
*Example: "A VLAN table (ID, name, purpose) and a segment diagram."*

**6. Detailed content.** The substance of the document. Structure is free-form beneath this heading, but all Section 6.1 writing rules apply.
*Belongs:* the actual content the document exists to carry. *Stays out:* anything already homed in parts 1–5 or in another document.
*Example: the VLAN table itself, the reasoning per segment.*

**7. Acceptance criteria.** Checkable statements defining "done."
*Belongs:* criteria answerable yes/no. *Stays out:* goals and intentions that cannot be checked.
*Example: "Every VLAN has an ID, a name, and a stated purpose. The diagram matches the table."*

**8. References.** Links to related documents and sources.
*Belongs:* pointers only — never copies. "None." if empty. *Stays out:* restated content from the referenced documents.
*Example: "Charter (audience) · Security Zone Model (zone definitions)."*

## 7. Acceptance criteria

- Every writing rule in 6.1 includes at least one example.
- The metadata block's fields, placement, and an example are defined in 6.2.
- Each of the eight parts in 6.3 states its purpose, what belongs, what stays out, and an example.
- The document points to the Change-Management SOP for metadata update rules and contains no update rules itself.
- The document itself complies with the format it defines.

## 8. References

- Project Charter — project purpose and canonical audience statement
- Change-Management SOP — rules for updating the metadata block *(not yet written; Task 3)*
- Master Document — where this Standard sits in the overall document map