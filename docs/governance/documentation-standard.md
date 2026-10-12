# Documentation Standard — defines how every document in this project is written.

**Status:** Approved · **Version:** 1.0 · **Last updated:** 2026-10-12 · **Owner:** Ioannis Mintzivyris

## 2. Overview

This document defines how every document in the project is written. It names the document types the project uses, and gives each type its format and lifecycle. It defines the metadata block, the eight-part format most types follow, and the writing rules that apply to every document.

A person who has never seen this project should be able to write a compliant document using this Standard alone.

## 3. Scope

Covers document types, format, structure, and writing rules.

Does not cover when and how document metadata is updated (see Change-Management SOP) or where documents live in the repository (see Master Document).

## 4. Dependencies

Requires the Project Charter — the project's purpose and audience are stated there and are not restated here.

## 5. Deliverables

- A table of document types, each with its format and lifecycle.
- The metadata block: its fields and where it sits.
- The eight-part format and what belongs in each part.
- The writing rules that apply to every document.

## 6. Detailed content

### 6.1 Document types

Every document in the project belongs to one of these types. The type decides its format and lifecycle. Lifecycle values (Draft, In Revision, Approved) are defined in the Change-Management SOP.

| Type | What it is | Format | Lifecycle |
|---|---|---|---|
| Controlled document | A policy, standard, design or procedure. States what should be true. | Metadata block and the eight parts. | Draft to Approved. Any change returns it to review. |
| Architecture Decision Record (ADR) | One decision, the options considered, and the reason for the choice. | Metadata block, then Context, Options considered, Decision, Consequences. | Approved once. Never edited; a later ADR supersedes it. |
| Record | Evidence of what happened, written at the time. | Metadata block and the eight parts. | Approved once. Later additions only, each dated. |
| Log | An append-only list of events. | Metadata block and the eight parts, with entries in a table. | Format approved once. Entries are appended without a new review and never edited. |
| Register | A maintained list of current items. | Metadata block and the eight parts. | Draft to Approved. Any change returns it to review. |
| Status report | Where the project stands, with evidence. | Date, then a table of phases with their status. | Updated in place. Every claim is dated, and every progress claim points to its evidence. |
| Lessons learned | What happened, why, and what changes next. | Title and date; free form. | Uncontrolled. Merged as written. |
| Reference material | Background and teaching material, not project facts. | Free form. | Uncontrolled. |

**README exemption.** The repository README is exempt from this Standard. It is the front page for arriving readers, carries no metadata block, and points to the status report.

### 6.2 Writing rules

These rules apply to every document of every type.

**Rule 1 — Point, don't copy.** Every fact has exactly one home. If information already lives in another document, link to it instead of restating it.
*Example: a design document needing the project's audience links to the Charter — it does not repeat the audience description.*

**Rule 2 — Explain jargon on first use.** Give the proper term plus a plain-language meaning the first time it appears. After that, use the term alone.
*Example: "The switch uses VLANs (virtual networks that separate traffic on shared hardware). Each VLAN is assigned..."*

**Rule 3 — Describe results, not steps.** Controlled documents, ADRs, registers and status reports state outcomes, not the actions taken. Records and logs are the exception: they state what happened, in order, as found.
*Example: a design document writes "Core switch segmented into three VLANs, isolating management traffic" — not "Logged into the switch, created VLAN 10, then VLAN 20..."*

**Rule 4 — Write for a mixed audience.** Readers range from recruiters to senior engineers. Plain English is the default; technical depth is welcome, but it must never be the only way to understand a sentence.
*Example: "Replication (how domain controllers keep their copies of the database in sync) is monitored daily."*

**Rule 5 — Keep it as short as the content allows.** Every sentence must earn its place. If a section says nothing, it says "None." in one line rather than padding.
*Example: a Dependencies section with no dependencies reads "None." — not a paragraph explaining why there are no dependencies.*

**Rule 6 — Name where every required record is kept.** A rule that requires something to be recorded names the document that holds the record.
*Example: "Every privileged session is recorded in the Privileged Access Log" — not "access is logged."*

**Rule 7 — Public by default.** The repository is public. No document contains a password, key, full serial number, MAC address (the fixed hardware address of a network port) or public IP address (an address reachable from the internet). Where an identifier is needed, only its last four characters are shown.
*Example: "Service tag ending MYQ2" — not the full tag.*

### 6.3 The metadata block

Every document begins with a metadata block, placed directly under the title and above the eight parts. It is a header, not a ninth part. The types that carry it are named in 6.1.

| Field | Meaning |
|---|---|
| Status | Where the document stands (e.g. Draft, Approved) |
| Version | The document's version number |
| Last updated | Date of the most recent change |
| Owner | Who is responsible for the document |

*Example:*
> **Status:** Draft · **Version:** 0.1 · **Last updated:** 2026-07-10 · **Owner:** Ioannis Mintzivyris

The rules for when and how these fields change are defined in the Change-Management SOP. This document defines only what the block contains and where it sits.

### 6.4 The eight parts

**Headings.** The title is the only H1 heading (the top-level heading). Parts 2 to 8 are H2 headings (the level below), numbered: `## 2. Overview` through `## 8. References`. Subsections of Detailed content are numbered 6.1, 6.2, and so on.

Every document that uses the eight-part format contains the following parts, in this order. The running example is a fictional VLAN Design document.

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

**6. Detailed content.** The substance of the document. Structure is free-form beneath this heading, but all Section 6.2 writing rules apply.
*Belongs:* the actual content the document exists to carry. *Stays out:* anything already homed in parts 1–5 or in another document.
*Example: the VLAN table itself, the reasoning per segment.*

**7. Acceptance criteria.** Checkable statements defining "done."
*Belongs:* criteria answerable yes/no. *Stays out:* goals and intentions that cannot be checked.
*Example: "Every VLAN has an ID, a name, and a stated purpose. The diagram matches the table."*

**8. References.** Links to related documents and sources.
*Belongs:* pointers only — never copies. "None." if empty. *Stays out:* restated content from the referenced documents.
*Example: "Charter (audience) · Security Zone Model (zone definitions)."*

## 7. Acceptance criteria

- Every document type in 6.1 has a description, a format and a lifecycle.
- The README exemption is stated.
- Every writing rule in 6.2 includes at least one example.
- The metadata block's fields, placement, and an example are defined in 6.3.
- The heading convention is stated in 6.4.
- Each of the eight parts in 6.4 states its purpose, what belongs, what stays out, and an example.
- The document points to the Change-Management SOP for metadata update rules and contains no update rules itself.
- The document names no specific tools.
- The document itself complies with the format it defines.

## 8. References

- Project Charter — project purpose and canonical audience statement
- Change-Management SOP — rules for updating the metadata block and the lifecycle values
- Master Document — where this Standard sits in the overall document map