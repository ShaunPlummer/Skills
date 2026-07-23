---
name: write-a-design-doc
description: Using the repository context, create a Technical Design Document for a proposed feature or engineering change. Use when the user asks to write a design doc, technical specification, implementation design, or architecture plan from requirements or conversation context. Ground the design in the current state of the codebase. Inspect the codebase before proposing changes, identify ambiguities and constraints, document user stories, BDD acceptance criteria, implementation decisions, testing strategy, exclusions, and unresolved questions, then save the result as Markdown under .tdd/.
metadata:
  author: Shaun Plummer
  version: "0.1.0"
---

## Process

1. Gather and reconcile the required information using the instructions below.
2. Once all material blocking questions are resolved and remaining assumptions are documented, complete the TDD template.
3. Create `<repo-root>/.tdd/` if it does not exist.
4. Write the TDD as a Markdown file named after the feature. If you know the task ID, use it to prefix the file name (e.g. `<repo-root>/.tdd/33050-user-onboarding.md`).
5. Once the file has been created, share its file name with the user.

## Information Gathering

Before completing the template:

- Ground the plan in the current codebase. Treat the implementation as the source of truth; verify the user's assertions and understand the
  relevant current behavior, architecture, conventions, assumptions, and constraints.
- Answer repository-verifiable questions through inspection instead of asking the user.
- Ask questions about unresolved decisions or ambiguities that prevent the template from being completed or affect the proposed solution.


### Reconcile the Conversation

Reconcile the conversation into a final decision set:

- Treat the user's later explicit decisions as superseding their earlier
  decisions.
- Treat assistant recommendations and proposals as undecided unless the user
  explicitly accepts them.
- Do not include superseded decisions as current requirements.
- Omit superseded decisions unless their rejection and rationale are important
  to understanding the final design.
- Surface contradictions that cannot be resolved from the conversation or
  repository.
- Preserve material negative decisions, constraints, and deliberately deferred
  work.

<tdd-template>

## Problem Statement

The problem that the user is facing, from the user's perspective.

## Solution

The solution to the problem, from the user's perspective.

## Assumptions

A numbered list of assumptions being made.

## Open Questions

### Blocking

A numbered list of questions that must be resolved before the design can be approved.

### Non-blocking

A numbered list of questions that can be resolved during planning or implementation without changing the agreed behavior.

## User Stories

A numbered list of user stories needed to describe the feature's externally observable behaviour. Each user story should be in the format:

1. As an <actor>, I want a <feature>, so that <benefit>.

<user-story-example>

1. As a reader, I want to see a list of today's news stories, so that I can stay up to date on current affairs.

</user-story-example>

This list of user stories should focus on the core high-value stories. Stories are extended with acceptance criteria. Developer user stories are only allowed if the TDD is being completed for a developer-facing product. For applications, implementation decisions describe technical constraints. Non-functional requirements should be expressed as measurable acceptance criteria where they relate to a specific user story. Non-functional requirements that affect multiple stories should be documented separately under Implementation Decisions.

## Story Details

### Story: <Story Number> - <Scenario Name>

A numbered list of any relevant acceptance criteria for the story, written in a BDD (Given, When, Then) format.

#### AC: <Story Number>.<Acceptance Criteria Number>

<story-details-example>

### Story: 1 - App Launch

#### AC 1.1

Given a user is logged in

When the home page loads

Then the personalised list of articles is displayed.


#### AC 1.2

Given a user is not logged in

When the home page loads

Then a sign-in prompt is displayed.


</story-details-example>

## Implementation Decisions

A list of implementation decisions that were made or proposed, grouped by section. Include only relevant subsections.

### Solution Diagrams

Include one or more diagrams using MermaidJS to provide a visual illustration of the relationship between components.

### General

* The project modules that will be added or modified.
* Architectural decisions.
* Technical clarifications from the developer.

### Data and Persistence Layer

* Data sources to be modified or new ones to be created.
* API contracts.
* Data storage mechanisms (file/key-value/SQL, etc.).

### Domain Layer

* Use cases to be modified or new ones to be created.

Do NOT include complete file paths or code snippets. They may become outdated very quickly. Class names and representations of the package structure are okay.

### Failure Handling

## Testing Strategy

Outline the testing approach for this feature, specifying what will be covered at each layer.

1. Unit tests should be added for all new classes.
2. Unit tests for existing classes should be updated to reflect the new functionality.
3. Tests should make use of a test scope DSL in order to group setup.

## Test Data
The range of data required for testing

<test-data-example>
* A journey summary with a single indoor leg representing an intra-building walk
* A journey summary with a single indoor leg representing an intra-cluster walk
* A journey summary with a transit leg
</test-data-example>

## Out of Scope

A numbered list describing things that are out of scope for this spec.

## Alternatives Considered

Any other major or significant design choices that were discarded.

## Further Notes

Any further notes about the feature.

</tdd-template>