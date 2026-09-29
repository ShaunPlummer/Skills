---
name: design-doc-to-plan
description: Converts an approved technical design document, PRD, specification, or architecture proposal into a phased implementation plan saved under ./.plans/. Use when turning a design doc or TDD into an implementation plan, vertical slices, or tracer-bullet phases.
---

# Design Doc to Plan

Break a provided design document (TDD) into a phased implementation plan using vertical slices (tracer bullets). Output is a Markdown file in `./.plans/`.

## Process

1. The design document should already be in the conversation. If it isn't, ask the user to share it.
2. Ground the plan in the current codebase. Treat the implementation as the source of truth; verify relevant architecture and clearly label assumptions.
3. Identify key architecture decisions on which the solution is being built.
4. Ask questions about unresolved decisions or ambiguities that prevent the template from being completed or affect the proposed plan.
5. Draft a series of vertical slices matching the provided rules.
5b. Anti-Pattern Self-Check:Inspect the drafted phases for the following anti-patterns before saving:❌ The Static UI Trap: Creating UI layout/mapping for dynamic status in Phase $N$ while deferring live stream collection (Flow.collect, reactive observers) to Phase $N+1$.❌ Untestable Increments: Any phase where a developer cannot visually observe the ACs working end-to-end on a live build because lower/higher layers aren't connected yet.❌ Dangling Mappers: Adding UI mapping code and static unit tests without connecting them to the screen's active ViewModel state pipeline.
6. Once you have a complete understanding of the problem and solution, use the template below to write the plan to a file. Create the `<repo-root>/.plans/` directory if it doesn't exist. Write the plan as a Markdown file named after the feature. If you know the task ID, use it to prefix the file name (e.g., `<repo-root>/.plans/33050-user-onboarding.md`).
7. Compare the newly created plan against the original design document to confirm no requirements or design decisions are missing.
8. Once the file has been created, share its file name with the user.

## Planning Rules

### Slice Shape

- Each phase should deliver a small, usable increment of functionality.
- Prefer thin vertical slices that cross the relevant application layers over horizontal, layer-by-layer work.
- Each phase must be independently demoable or verifiable.
- Keep phases small enough to integrate, review, and receive feedback quickly.
- A story may span multiple phases, with each phase delivering a distinct subset of its acceptance criteria.
- Never leave comments in the code referencing phases of the implementation plan. You must only ever document the current state of the codebase.

### UI and Data Stream Coupling
- **Never split UI display wiring and the underlying reactive data path into separate phases.** 
- If a phase introduces or updates a UI surface to render dynamic state (e.g., Scheduled / Delayed / Cancelled, status badges, overlays):
  - The phase **MUST** wire the real reactive data source (`Flow`, `LiveData`, observer, etc.) into the ViewModel or UI controller in that same phase.
  - A phase cannot land UI layout/mapping for dynamic states while relying solely on static, one-shot, or mock/plan-cache data sources if the design calls for live updates.
- If an underlying SDK dependency, endpoint, or reactive method (e.g., `observeJourneySummary`) is required to drive the UI state, pin the dependency and wire the reactive pipeline in the **same phase** as the UI update.

### Just-in-time introduction (no forward scaffolding)

Do **not** add types, enum entries, config keys, DI wiring, fixtures, or mock updates in an early “foundations” phase merely because a later phase will need them.

- Introduce new symbols in the first phase whose acceptance criteria or Verification require it.
- If a phase only needs a subset (e.g. DTO fields for deserialize), do not add unused domain models, repository APIs, endpoint/feature enums, or mock config properties “for later”.
- Put deferred symbols in that phase’s **Phase out of scope** / Implementation Details deferred list by name (e.g. “`EndpointType.REGISTER_REALTIME` — Phase 3”).
- Enabling work is allowed only when **this phase’s** ACs cannot be verified without it (e.g. case-insensitive config lookup can be tested with **existing** endpoint/feature ids; do not invent new enum values until a phase resolves those ids).
- Prefer verifying behaviour with the smallest surface: DTO-only → domain when mapped → repository when saved → config keys when the network/API path reads them.

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

## Phase implementation
- Never leave code comments mentioning the phases

<plan-template>
# Implementation Plan: <Feature/Design Doc Name>

## Problem Statement

The problem that the user is facing, from the user's perspective.

## Solution

The solution to the problem, from the user's perspective.

## Phases

| Phase | Goal | Outline |
| -- 	| --	| --  |
|Phase Name | Key Aims | Description of changes |

<phase-template>
## Phase 1: <Title>

A description of the phase. Including the part of the problem it attempts to address as well as the goals and outcomes of the phase.

### Story: <Story Number> - <Scenario Name>

A numbered list of any relevant acceptance criteria for the story, written in a BDD (Given, When, Then) format.

#### AC: <Story Number>.<Acceptance Criteria Number>

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
- There is a unit test for every BDD scenario implemented in this phase
- **End-to-End / Interactive Verification:**
  - Explicitly detail how this slice will be verified in a running application session (e.g., "Start navigation, trigger SDK overlay poll, observe Summary UI dynamically updates from Scheduled to Delayed without re-opening the screen").
  - **Verification Gate:** If a phase cannot be validated in a real user flow because the live data pipeline is missing, the slice is invalid and MUST be merged with the data wiring phase.
  
#### Testing Data

### Phase out of scope

A numbered list of acceptance criteria which are part of the user story included in this phase which will be completed in another phase.

</phase-template>

## Further Notes

Any further notes about the feature.

</plan-template>