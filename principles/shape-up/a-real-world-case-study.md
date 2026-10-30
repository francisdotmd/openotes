---
title: "Shape Up: A Real-World Case Study"
description: A teaching walkthrough of Shape Up, following a gym-payment project from first idea through launch.
tags:
  - shape-up
  - product-development
  - case-study
source_url: https://www.youtube.com/watch?v=Holk0GsGYfY
---
# Learn Shape Up through a real project

Let’s use this case to understand how Shape Up works. The project begins with a vague request—“improve the dashboard”—and ends with a focused effort to help gym owners recover failed member payments. As you read, watch how the team reduces uncertainty before asking the build team to take on the work.

## The basic idea

Treat the team’s build time as valuable. Before the cycle starts, get clear on two things:

1. **Frame the problem:** What is going wrong for the customer, and what valuable outcome should change?
2. **Shape a solution:** What are we going to build, and does the team understand enough to build it without major surprises?

Then give the team a shared picture of the work and break it into pieces that can each become demonstrable progress. The point is not to plan every keystroke; it is to make the purpose and the important parts of the solution clear enough that the team can work with fewer interruptions and still finish.

## Terms you’ll see in the case

- **Candidate:** a possible next project. “Improve the dashboard” is a candidate, not yet a clear problem or plan.
- **Frame:** the customer problem and the outcome worth achieving. Ask: *Why is this worth doing?*
- **Shape:** a workable design for the important flows, including enough technical understanding to see likely complexity. Ask: *What are we building, and how can it work?*
- **Pitch:** a written reference that carries the agreed frame and shape into the team kickoff.
- **Scope / vertical slice:** a small, demonstrable part of the product that connects the necessary backend and interface work. A backend-only layer is not yet a usable slice.
- **Cycle:** the team’s fixed build window. The six weeks mentioned in the case is an example, not a rule for every team.

## Follow the project from idea to launch

### Decide what “good” means before building

Start with two expectations: build time is precious, and when you commit to a project you want to reach a finish line. That rules out both open-ended projects and activity that looks like progress but never produces something you can show. Framing and shaping are how you give the team a clearer path to a valuable, finishable result.

### Adapt the method to your team

The case contrasts a small, senior, highly overlapping team with a more specialized team that has separate backend, frontend, design, and QA roles. The lesson for you: copy the clarity-building, not another company’s exact org chart. Your process should bring the right knowledge together, then leave people room to do their work.

### Treat “improve the dashboard” as a clue

Leadership suggests improving a gym-management dashboard. Don’t jump straight into tickets: “improve” does not tell you what is wrong, what better looks like, or how to choose between options. Instead, ask who understands the customer and the history behind the request. That investigation turns a blurry candidate into something you can frame.

### Find the customer value

The subject-matter expert explains that small gym owners already know their new members and recent sales. Those dashboard numbers add little. His old spreadsheet points to missed recurring payments—a problem that matters to the owner but is buried in the product’s reports. Class utilization also comes up as a potentially useful metric. The frame is now more specific: show information that helps a small gym owner run the business, rather than merely “make the dashboard better.”

### A good frame still needs a shape

It may sound easy to surface existing data, but hidden technical or product complexity can derail a cycle. Before building, ask what the solution will actually require and how it will work. Bring in an engineer who knows the product and codebase to explore those questions with you.

### Sketch the flow and look for hidden edges

The engineer and product lead sketch dashboard panels, then trace where failed-payment information lives across contracts, invoices, sales, customers, and members. They discover that the existing sale-detail page is legacy code handling several types of sales, so adding payment status and retry actions is not a simple surface change. They also uncover unanswered cases: repeated failures, automatic retries, card updates, and the time range needed for class-utilization trends. This is shaping at work: finding the questions that would otherwise appear halfway through the build.

### Let new evidence narrow the project

A follow-up with the expert reveals that the product cannot properly retry some failed payments or help owners request a new card. The valuable outcome shifts from *show missed payments* to *help recover them*. Class utilization could still be useful, but it is not essential to this project and is dropped. Notice the discipline: the team does not keep adding desirable ideas just because they came up. It protects the strongest outcome.

### Get the right people in one shaping session

The product lead brings the engineer and expert together to resolve the remaining design questions live. They revise the solution until it makes sense for the customer and is technically workable. That shared confidence is the “shape go” moment: the team is ready to build. The project does not need a fresh popularity vote here because the value was already aligned during candidate selection and framing.

### Use the pitch to transfer clarity

Now write down the frame and shape so the whole team has a reference at kickoff. The document captures the problem, the designed flows, and a breadboard—a rough map of screens and interactions. Use kickoff to walk through that shared picture and identify scopes. The case uses vertical slices that can be built and demoed, and sequences them so working progress appears early; the suggested limit of nine scopes is a practice from this example, not a universal rule.

### Make it work, then make it polished

The team connects frontend and backend into an ugly but functioning prototype, tests the flows, and applies visual polish afterward. Because the team has a clear direction, it can work asynchronously for long stretches and share progress without constant status meetings. The project finishes with working scopes and a launch brief showing the completed experience.

## What this case teaches you

1. **Don’t build from a vague request.** Ask what customer problem the request points to.
2. **Separate the “why” from the “how.”** Frame the value; shape a feasible way to deliver it.
3. **Invite the people who know the customer and system.** Their knowledge exposes missing requirements and hidden complexity.
4. **Use discovery to cut scope as well as add detail.** Payment recovery mattered more than adding every useful dashboard metric.
5. **Write the pitch to share a decision, not replace the thinking.** The conversations and sketches created the clarity; the document helped carry it to kickoff.
6. **Build demonstrable slices before polish.** Get a real flow working, then refine its appearance.

The case’s specific cycle length, meeting format, and number of scopes are examples. Keep the principle—clarity before the build, then focused progress—and adapt the mechanics to your team.

## Remember it this way

**Choose a worthwhile problem → understand the value → shape a feasible solution → build it in demonstrable slices → finish and ship.**

---

## Open Questions

1. We got all the questions, the answers, then framed the to the right problem, but why they didn't comeback to top (to the candidates, or whomever is available) to get aligned from what the problem truly is? \
   Because for me, it doesn't mean that you're the CPO (Chief Product Officer) that you solely handles the product itself.\
   And Singer said that, these candidates will be happy whatever they had within the process.
   <img src="/principles/shape-up/a-real-world-case-study.assets/pasted-20261030-015534.png" />
2. (Not a question but I observed) They build it first in a wireframe, an ugly UI but every functions works, then they paint the wireframe. It's like building a house where in structure first, pipe, electrical wires and whatnot will be built first then paint afterwards.
3. I noticed here that framing is one of the important process, it involves all the expensive people but what I noticed here is why there's none when it comes to shaping? People just came back after the CPO shaped (?) the project which is at the kickoff.
   <img src="/principles/shape-up/a-real-world-case-study.assets/pasted-20261030-021447.png" />

