---
name: tdd-to-plan
description: Use when converting an approved technical design document, PRD, specification, or architecture proposal into a phased implementation plan. Saved as a local Markdown file in ./.plans/.
metadata:
  author: Shaun Plummer
  version: "0.1.0"
---

# TDD to Plan

Break a provided TDD into a phased implementation plan using vertical slices (tracer bullets). Output is a Markdown file in `./.plans/`.

## Process

1. The TDD should already be in the conversation. If it isn't, ask the user to share it.
2. Ground the plan in the current codebase. Treat the implementation as the source of truth; verify relevant architecture and clearly label assumptions.
3. Identify key architecture decisions on which the solution is being built.
4. Ask questions about unresolved decisions or ambiguities that prevent the template from being completed or affect the proposed plan.
5. Draft a series of vertical slices matching the provided rules.
6. Once you have a complete understanding of the problem and solution, use the template below to write the plan to a file. Create the `<repo-root>/.plans/` directory if it doesn't exist. Write the plan as a Markdown file named after the feature. If you know the task ID, use it to prefix the file name (e.g., `<repo-root>/.plans/33050-user-onboarding.md`).
7. Compare the newly created plan against the original TDD to confirm no requirements or design decisions are missing.
8. Once the file has been created, share its file name with the user.

## Planning Rules

### Slice Shape

- Each phase should deliver a small, usable increment of functionality.
- Prefer thin vertical slices that cross the relevant application layers over horizontal, layer-by-layer work.
- Each phase must be independently demoable or verifiable.
- Keep phases small enough to integrate, review, and receive feedback quickly.
- A story may span multiple phases, with each phase delivering a distinct subset of its acceptance criteria.

### Risk and Enabling Work

- Prioritise phases that reduce the greatest risk or uncertainty.
- Add a test harness before changing unclear existing behaviour.
- Introduce seams or refactor only when required by the next functional phase.
- A phase may contain only refactoring to separate it from the functional change.
- Use a time-boxed spike when uncertainty prevents reliable planning. Define the question being investigated and the spike’s exit criteria.

## Plan Validation

- Copy story, acceptance-criterion, and decision IDs unchanged from the design.
- Identify the stories and acceptance criteria addressed by every phase.
- Map every in-scope acceptance criterion to at least one phase.
- Do not introduce product behaviour that is absent from the design.
- Do not include items identified as out of scope.
- Follow the design’s technical decisions. If repository evidence suggests a decision should change, identify the conflict and return it for review rather than silently changing it.

<plan-template>
# Implementation Plan: <Feature/TDD Name>

## Problem Statement

The problem that the user is facing, from the user's perspective.

## Solution

The solution to the problem, from the user's perspective.

<phase-template>
## Phase 1: <Title>

A description of the scope and outcome for the phase.

### <Story Number> - <Scenario Name>

A numbered list of any relevant acceptance criteria for the story, written in a BDD (Given, When, Then) format.

#### <Story Number>.<Acceptance Criteria Number>

### Implementation Details

- The behavior introduced by this phase.
- The relevant components or modules to change.
- The technical approach and important sequencing.
- Any migrations, compatibility considerations, or feature flags.
- Constraints inherited from the design document.
- Work deliberately deferred to later phases.

### Verification

- Unit tests must be added for new functionality.
- Unit tests must be updated where existing functionality is modified or extended.
- Confirm `./gradlew check` passes.
- Manual steps needed to demonstrate the slice.

#### Testing Data

### Phase out of scope

A numbered list of acceptance criteria which are part of the user story included in this phase which will be completed in another phase.

</phase-template>

## Further Notes

Any further notes about the feature.

</plan-template>