# Where Tool Names Live — one home for tooling

## The decision

When writing the Documentation Standard, I agreed a rule: **this document names no specific tools.** It describes *what* is done in tool-agnostic terms. The record of *which* tools are used lives in one place only — the Master Document.

## Why

This is my "point, don't copy — each fact has one home" principle, applied to tooling. If every document named its own tools, then the day I switch a tool (say, swap one editor or platform for another), I'd have to hunt through every document and update each mention. Things would drift; some would get missed; the docs would start contradicting reality.

Instead, tool names have a single home. The Documentation Standard says *what happens*; the Master Document says *what it's done with*. Change a tool, and I update exactly one place.

## The wider lesson

A document describing a *process* is more durable when it's separated from the *implementation* of that process. The process ("changes are proposed, reviewed, and merged") outlives any particular tool that carries it out. Keeping tool names out of the process document means the process document doesn't age every time the toolchain changes.

## One-line takeaway

Name tools in exactly one place (the Master Document). Process documents describe *what*, not *which tool* — so they don't rot when tools change.