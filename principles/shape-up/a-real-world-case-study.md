---
title: "Shape Up: A Real-World Case Study"
description: A plain-language walkthrough of Ryan Singer’s gym-payment project, from candidate selection through launch.
tags:
  - shape-up
  - product-development
  - case-study
speaker: Ryan Singer
source_url: https://www.youtube.com/watch?v=Holk0GsGYfY
---
# Shape Up: A Real-World Case Study

## The idea in one sentence

This guide follows Ryan Singer’s video walkthrough. Its URL is recorded in this document’s `source_url` metadata and in the linked [transcript](../../external-sources/shape-up-real-world-case-study-transcript.md).

Shape Up is a way to spend a fixed cycle on a valuable, finishable project by clarifying the customer problem first, working out a feasible solution before the team starts, then building in independently demoable slices. Singer’s case moves from “improve the dashboard” to recovering failed gym-member payments. [Transcript](../../external-sources/shape-up-real-world-case-study-transcript.md)

## A quick map of the terms

- **Candidate:** a possible next project leadership thinks may be worth doing. It is not yet a clear brief.
- **Frame:** explain the customer/business problem and the outcome worth achieving. The frame answers “why this?”
- **Shape:** work out a feasible solution and its important flows before committing the build team. The shape answers “what are we building, and how can it work?”
- **Pitch:** the written reference that captures the frame and shape for the team. In this case it is prepared after the shaping decision, mainly to support kickoff—not as one item in a large pitch contest.
- **Scopes / vertical slices:** chunks that each connect enough backend and interface work to demonstrate a real part of the product. They are sequenced so the team can see working progress.
- **Cycle:** the fixed period the team is working within. Singer uses six weeks as an example, not as a universal requirement.

The two assumptions behind the talk are that engineering time is precious and that a chosen project should reach a finish line within its timebox—not remain an open-ended effort or a collection of progress that never becomes demoable. [Transcript](../../external-sources/shape-up-real-world-case-study-transcript.md)

## Walkthrough in video order

**Timing note:** the transcript you provided has no timestamps. These are rough chapter locations inferred from its sequence, not verified minute marks; use them to find the section, not to quote an exact time. [Transcript](../../external-sources/shape-up-real-world-case-study-transcript.md)

### Why frame and shape before building?

There are separates two kinds of uncertainty. *Framing* gets clear on the problem and customer value. *Shaping* gets clear on a workable technical/product solution. Both come before the build so the team does not spend its cycle discovering basic purpose or hidden complexity. The goal is valuable work that finishes, with enough clarity for the team to work independently and avoid unnecessary coordination. [Transcript](../../external-sources/shape-up-real-world-case-study-transcript.md)

### \~05:00–08:00 — Shape Up is a practice, not a universal org chart

Singer explains that the book describes how Basecamp worked, and that Basecamp had a small, senior, technically overlapping team. This case is at a different company with separate backend, frontend, design, QA, sales, and marketing responsibilities. The useful question is not “copy Basecamp exactly,” but “how do we make the problem and solution clear enough for this team?” [Transcript](../../external-sources/shape-up-real-world-case-study-transcript.md)

### \~08:00–12:00 — Start with a candidate, not a solution

Leadership proposes “improve the dashboard.” That is a candidate, not yet a buildable project: it does not say what is wrong, what “better” means, or how to make tradeoffs. The CPO asks who knows the backstory and finds a sales subject-matter expert who has run a gym. This is detective work to make the candidate frameable—not a reason to reject it. [Transcript](../../external-sources/shape-up-real-world-case-study-transcript.md)

### \~12:00–17:00 — Frame around the gym owner’s real problem

The acquired product’s dashboard shows new members and recent sales, but small gym owners already know those people and transactions. The expert’s old spreadsheet reveals a more useful concern: missed recurring payments. The information exists in the product, but is buried in reports and filters. Class utilization is another possible metric. The frame gets sharper: focus on useful small-gym metrics, rather than “make the dashboard better.” [Transcript](../../external-sources/shape-up-real-world-case-study-transcript.md)

### \~17:00–20:00 — Don’t mistake a clear problem for a shaped solution

“Just surface the data” sounds small, but Singer expects hidden complexity to appear during a cycle if no one investigates it first. The next checkpoint is *shape*: can the team explain what it will take to build the solution and how it will work? He pairs with a senior engineer who knows the product and codebase to explore that. [Transcript](../../external-sources/shape-up-real-world-case-study-transcript.md)

### \~20:00–26:00 — Shape with the people who know the system

In a whiteboard session, they sketch the dashboard panels and trace where failed-payment data lives across contracts, invoices, sales, customers, and members. A row in a panel needs a useful detail view; the existing sale-detail page is legacy code handling several kinds of sales. Adding payment status and retry actions there may be harder than it first seemed. The session surfaces unanswered product questions too: retries, repeated invoice failures, card updates, and how much history a class-utilization trend needs. The point of shaping is to expose these questions before the build is underway. [Transcript](../../external-sources/shape-up-real-world-case-study-transcript.md)

### \~26:00–30:00 — New evidence changes the frame

The subject-matter expert clarifies that the product cannot properly retry some failed payments or help an owner request a new card. The valuable outcome is therefore not merely *show* missed payments; it is *help recover* them. Class utilization could be useful, but it was raised only because the CPO asked for more ideas. The team drops it from this project and narrows the frame to payment recovery. This is scope discipline: one strong outcome beats a broader dashboard bundle that may not fit. [Transcript](../../external-sources/shape-up-real-world-case-study-transcript.md)

### \~30:00–34:00 — Resolve open questions together; reach “shape go”

The CPO brings the engineer and subject-matter expert into one more whiteboard session. They revisit the unresolved cases and adjust the design until it makes sense for users and is technically workable. That shared conclusion is the shape checkpoint: the team can say “this is something we can go build.” The alignment was already established while choosing and framing the work, so this is not another late-stage popularity vote. [Transcript](../../external-sources/shape-up-real-world-case-study-transcript.md)

### \~34:00–38:00 — Write the pitch, then plan vertical slices

After the solution is shaped, the CPO writes the pitch as a source of truth for kickoff: it summarizes the frame, the solution, and the breadboard (a rough map of screens and flows). The whole team meets, reviews it, and identifies a small set of scopes—Singer says a maximum of nine works well for him. Each scope is a vertical slice that can be built and demoed, rather than a layer such as “all backend” with nothing usable yet. They also order the slices so working progress appears early. [Transcript](../../external-sources/shape-up-real-world-case-study-transcript.md)

### \~38:00–end — Build the working thing before polishing it

The team works asynchronously for stretches because the kickoff and shaping provide shared direction. They start from the breadboard, connect frontend and backend, and get an unattractive but functioning prototype that can be tested. Visual polish comes after the flows work. The project ends with finished scopes and a launch brief showing the result. Singer’s closing lesson is progressive investment: move from a broad candidate to a clear frame, then a feasible shape, then a build. In his example, concentrated live sessions upstream create the clarity that lets the team spend much of the build phase working independently. [Transcript](../../external-sources/shape-up-real-world-case-study-transcript.md)

## What to remember

1. **A request is not yet a problem statement.** “Improve the dashboard” needed customer context before it could guide tradeoffs.
2. **Frame for value; shape for feasibility.** The gym-owner evidence explained both why the work mattered and what the solution needed to do.
3. **Bring expertise into the shaping conversation.** The engineer exposed system constraints; the subject-matter expert resolved domain questions.
4. **Let discovery narrow scope.** The project moved from dashboard metrics to payment recovery, and left class utilization out.
5. **Write the pitch to transfer clarity.** In this case it records an already-shaped project for kickoff; the document is not a substitute for the conversations that produced the design.
6. **Demo vertical progress, then polish.** A working slice gives the team something real to test and show before visual finish.

These are lessons from this example, not a claim that every team must use the same meeting count, scope limit, or cycle length. [Transcript](../../external-sources/shape-up-real-world-case-study-transcript.md)

## One-line mental model

**Choose a worthwhile problem → understand the value → design a feasible approach → give the team clear slices → finish and ship.**