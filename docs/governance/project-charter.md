# Project Charter — itc-hybrid-prj (Hybrid Homelab Project)

## Overview

This document is the charter for **itc-hybrid-prj** (the Hybrid Homelab Project). It explains why the project exists, its objectives, how success is measured, and the constraints and sponsor parameters that guide it. It is the reference point for decisions about scope and priority during the whole project.

This is a public repository. The readers are the project owner and anyone who may be interested — professional contacts, technical recruiters, and interviewers. A reader should be able to understand the reason for the project and its limits without reading all the technical documents.

---

## Scope

**In scope for this charter:**
- The purpose and objectives of the project.
- The success criteria that define when the project is meeting its goals.
- The main constraints (time, budget, tools, environment).
- The sponsor parameters: the owner's hourly rate and total-effort estimate.

**Out of scope for this charter:**
- Technical designs and architecture. These live in the `docs/architecture/` and `docs/design/` folders.
- Step-by-step procedures for building or recovering systems. These live in `docs/procedures/`.
- Decision records that explain specific technical choices. These live in the `ADR/` folder.

This charter is the top-level "why" document. All "how" content lives in the technical documents it points to.

---

## Dependencies

This charter builds on:
- **Master Document** — the main guide that sets the project's structure, workflow, and rules.
- **Documentation Standard (8-part format)** — the format this charter is written in. (This formal standard is the next task to be written; for now, the format comes from the Master Document.)
- **Valuing Your Time (companion document)** — the source of the owner's hourly rate and effort estimate, used later in Detailed Content.
- **The owner's time** — the charter needs the owner's effort to be finished and kept up to date.

---

## Deliverables

When this task is complete, the following exists:
- **A finished project charter** at `docs/governance/project-charter.md`, written in the 8-part format.
- **A clear statement of purpose, objectives, and success criteria** that a reader can understand without any other document.
- **Recorded sponsor parameters** — the owner's hourly rate and total-effort estimate — giving the project an agreed value in time and money.

---

## Detailed Content

### Purpose

This project exists to build and document a realistic hybrid IT environment — on-premises infrastructure combined with Azure cloud — that serves as clear, professional evidence of hands-on infrastructure engineering ability. The owner brings a strong foundation in senior IT support and administration (L1–L3), with a consistent track record of solving complex problems across networks, Azure, and applications — often well beyond the formal scope of the role. This demonstrates a core strength: the ability to move into new technical areas quickly and deliver results. This project builds directly on that strength, turning proven adaptability and hands-on knowledge into visible, verifiable evidence of the ability to design, build, run, and document hybrid infrastructure — supporting a move into a Hybrid Infrastructure Engineer role spanning on-premises and Azure cloud.

### Objectives

The project sets out to:
1. **Build a working hybrid infrastructure** — a functioning on-premises environment (identity, networking, compute, security) integrated with Azure cloud.
2. **Demonstrate senior-level design thinking** — make and record deliberate architectural decisions, not just follow steps.
3. **Produce professional documentation** — maintain clear, standardised documentation and governance that a stranger could follow and trust.
4. **Practise real engineering discipline** — use proper version control, change management, and a structured workflow throughout.
5. **Create visible, verifiable evidence** — leave a public record that proves the ability to design, build, run, and document hybrid infrastructure.

### Success Criteria

The project is succeeding when the following are true — each stated as a result that can be checked, not a task that was done:

1. **The environment works end to end.** A user can authenticate on-premises and against Azure, across the integrated hybrid setup — demonstrably, not just "installed."
2. **Every major design choice is recorded and justified.** Each significant decision has a written record explaining what was chosen and why (an ADR or equivalent).
3. **A stranger can navigate the project unaided.** Someone new can open the repository and understand what it is, why it exists, and how it is organised — without help.
4. **The work is traceable.** The Git history and Kanban board show a clean, structured workflow: one task per branch, standard commits, a clear path from idea to done.
5. **The evidence stands on its own.** The public repository, read cold by a technical recruiter, communicates the owner's ability to design, build, run, and document hybrid infrastructure.

**Guiding principle — Achievements over tasks:** Throughout this project, all work is measured and documented by the result it achieves, not the steps taken. Every completed piece answers "what does this now make possible?" — not merely "what did I do?" This principle exists to ensure the project produces visible, tangible outcomes, and to close the gap between real skill and recorded evidence.

### Constraints

The project operates under the following limits:

1. **Time — currently favourable but variable.** The project currently benefits from a period of concentrated availability, allowing steady, focused progress. This availability may reduce at short notice if new employment begins. The project is therefore structured in self-contained, incremental pieces, so it can absorb a change in pace without losing coherence.
2. **Resources.** It runs on available lab hardware and Azure resources within a personal budget, which sets practical limits on scale (number of hosts, cloud spend, and so on).
3. **Single operator.** One person fills every role — owner, engineer, documenter. There is no team to divide work or cross-check it, so discipline and documentation carry that load.
4. **Scope discipline.** The project deliberately limits itself to what serves its purpose. Interesting-but-unrelated technologies are kept out (recorded in the backlog if worth revisiting) to avoid scope creep.
5. **Knowledge in progress.** Some areas are being learned as the project proceeds. This is expected and intended — the project is a learning vehicle — but it means some early work may be revised as understanding deepens.

### Sponsor Parameters (Project Value)

**Sponsor parameters** are the figures set by the project's sponsor — the person who backs and funds it. For this project, the owner is the sponsor. Two figures define the project's value in time and money.

**Hourly rate: SEK 290/hour.**
This is the entry-level market rate for a Hybrid Infrastructure Engineer in the Stockholm area — the role this project supports a move into. It values the owner's time at the honest, current market rate for the target role, rather than an inflated figure.
*Note:* SEK 290/hour is an employment-equivalent rate (gross salary divided by working hours). The same work valued as freelance or consultancy would be roughly 2–3× higher (approximately SEK 600–900/hour), because a consultant must self-fund social contributions, pension, vacation, insurance, and downtime. The employment rate is used here because the project's goal is employment, not consultancy.

**Total-effort estimate: approximately 350 hours.**
This is built from the bottom up — estimating each project phase (governance, identity, PKI, federation and cloud, compute, network, security, monitoring and recovery, and ongoing documentation) and summing them to roughly 270 hours, then adding a 30% contingency buffer for learning curves, troubleshooting, and rework. The buffer reflects that some areas are being learned as the project proceeds (see Constraints).

**Project value: approximately SEK 100,000** (≈ 350 hours × SEK 290/hour).
This figure represents the value of the time and effort invested. It is stated to make the project's cost visible and real, to treat the work with the seriousness of a professional engagement, and to demonstrate estimation as a practised skill.

---

## Acceptance Criteria

This charter is considered complete when all of the following are true:

1. **All eight standard sections are present and filled in** — Title, Overview, Scope, Dependencies, Deliverables, Detailed Content, Acceptance Criteria, and References.
2. **The purpose is clear to a stranger.** A reader with no prior knowledge can understand why the project exists after reading the Purpose alone.
3. **Success is defined and checkable.** The Success Criteria state, in tickable terms, what "success" means for the project.
4. **The sponsor parameters are set and reasoned.** Both the hourly rate and the effort estimate are present, with the reasoning behind them, not just the numbers.
5. **Scope boundaries are explicit.** The document states both what it covers and what it deliberately does not.
6. **The language is accessible.** A non-specialist reader (for example a recruiter) can follow the document without needing deep technical knowledge; business terms are explained on first use.
7. **The document stands alone.** It can be read and understood without any other document open alongside it.

---

## References

**Internal (project documents):**
- **Master Document** — the project's single source of truth for structure, workflow, and rules. This charter operates within its framework.
- **Documentation Standard** — the 8-part format this charter follows. *(To be written as the next Phase 0 task; until then, the format is defined in the Master Document.)*
- **Valuing Your Time (companion document)** — the reasoning behind the sponsor parameters (hourly rate and effort estimate).
- **Change-Management SOP** — the procedure governing how changes are made safely. *(To be written later in Phase 0; referenced here as the mechanism for any future changes to this charter.)*

**External (sources used):**
- Swedish IT salary market data (Stockholm entry-level infrastructure/systems engineer rates, 2026), used to set the hourly rate. Sources include ERI SalaryExpert and comparable Swedish salary surveys.