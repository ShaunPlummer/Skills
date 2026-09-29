---
name: write-a-design-doc
description: Creates a Technical Design Document grounded in the current codebase, covering user stories, BDD acceptance criteria, implementation decisions, testing strategy, and open questions, then saves it under .tdd/. Use when the user asks to write a design doc, technical specification, implementation design, or architecture plan.
---

## Process

1. Complete 'Information Gathering' and 'Reconcile the Conversation' steps using the instructions below.
2. Once all material blocking questions are resolved and remaining assumptions are documented, complete the TDD template.
3. Create `<repo-root>/.tdd/` if it does not exist.
4. Write the TDD as a Markdown file named after the feature. If you know the task ID, use it to prefix the file name (e.g. `<repo-root>/.tdd/33050-user-onboarding.md`).
5. Complete the 'Review and Revision' instructions below.
6. Once the file has been created, share its file name with the user.

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
- Put verified facts that inform the design under Key Considerations.
- Put required behaviour under acceptance criteria or relevant implementation section.
- Put selected technical approaches under Implementation Decisions.

### Review and Revision

- Review all sections of the document for consistency
- Review the document for duplicated information that can be removed. 

<tdd-template>

> This is a living design document that reflects the current intended end state and will be updated as the design evolves. It does not describe or track implementation progress or completed work.


## Problem Statement

The problem that the user is facing, from the user's perspective.

## Solution

The solution to the problem, from the user's perspective.

## Key considerations

A numbered list of requirements and constraints on which the design should be based. For each item state why the design must take it into consideration. This section should outline inputs to the design and not include implementation decisions made in response to the constraints. These are documented later in the template.

This section may include repository verified facts that must not be changed as part of this design.

## Assumptions

A numbered list of unverified conditions the design relies on. For each assumption, include how they will be validated and what changes if they proven false.

## Risks

A numbered list of major risks which have been identified and any mitigations which can be implemented.

## Dependencies

A numbered list dependencies. This can include external work or other development efforts.

## Open Questions

### Blocking

A numbered list of questions that the user must be resolved before the design can be approved.

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

#### AC 1.1 Personalised Adds are displayed for authenticated users

	Given a user is logged in

	When the home page loads

	Then the personalised list of articles is displayed.


#### AC 1.2 unauthenticated users are prompted to sign in

	Given a user is not logged in

	When the home page loads

	Then a sign-in prompt is displayed.


</story-details-example>

## Implementation Outline

Explain how the proposed solution works in a series of connected paragraphs. Describe the intended end state, the changes to existing behaviour, and how the affected components interact. Clearly outine relationships between UI, domain and data layers of the application.

Provide enough detail for a reader to understand the solution without reconstructing it from the implementation decisions. Focus on responsibilities, interactions, and data flow. Include only relevant subsections, adapting the headings to the system's architecture.

Do NOT include complete file paths. Do include class names, representations of the package structure. You may include concise contract, schema, or pseudocode examples.

### Solution Diagrams

Include one or more diagrams using MermaidJS to provide a visual illustration of the relationship between components.

### Components and Responsibilities

Describe the components that will be added or modified, their responsibilities, and how they interact.

### Data and Interfaces

Describe relevant data sources, API contracts, storage mechanisms, and the flow of data through the application layers. Explain changes to existing components and persisted data.

### Domain Behaviour

Describe the use cases and business logic being introduced or changed. Consider happy and error path behaviour.

## Implementation Decisions

A numbered list of significant implementation decisions made in response to the requirements and considerations that affect the final design. Include decisions where a alternative existed but was not selected.

For each decision capture the choice, why it was selected and any significant trade-offs or consequences of this decision. Avoid repeating the implementation outline. Each item should be concise.

### Failure Handling

A numbered list of possible failure scenarios and a description of how the design responds when failures do occur.

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

A numbered list describing functionality, problems or modifications that this design deliberately does not change.

## Alternatives Considered

Describe alternative approaches to solving the problem statement which were not selected. This section must not make assumptions about why the user rejected the solution. Only include reasons why they explicitially tell you why it was rejected. 

## Further Notes

Any further notes about the feature.

</tdd-template>