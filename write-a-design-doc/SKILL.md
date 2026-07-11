---
name: write-a-design-doc
description: Using the repository context, create a Technical Design Document for a proposed feature or engineering change. Use when the user asks to write a design doc, technical specification, implementation design, or architecture plan from requirements or conversation context. Ground the design in the current state of the codebase. Inspect the codebase before proposing changes, identify ambiguities and constraints, document user stories, BDD acceptance criteria, implementation decisions, testing strategy, exclusions, and unresolved questions, then save the result as Markdown under .tdd/.
metadata:
	author: Shaun Plummer
	version: "0.1.0"
---

## Process

1. Explore the repo to verify the user's assertions and understand the current state of the codebase.
2. If a question can be answered by exploring the codebase, explore the codebase instead.
3. Ask questions about unresolved decisions or ambiguities that prevent the template from being completed or affect the proposed solution.
4. Once you have a complete understanding of the problem and solution, use the template below to write the TDD to a file. Create the `<repo-root>/.tdd/` directory if it doesn't exist. Write the TDD as a Markdown file named after the feature. If you know the task ID, use it to prefix the file name (e.g. `<repo-root>/.tdd/33050-user-onboarding.md`).
5. Once the file has been created, share its file name with the user.

<tdd-template>

## Problem Statement

The problem that the user is facing, from the user's perspective.

## Solution

The solution to the problem, from the user's perspective.

## User Stories

A numbered list of user stories needed to describe the feature's externally observable behaviour. Each user story should be in the format:

1. As an <actor>, I want a <feature>, so that <benefit>.

<user-story-example>

1. As a reader, I want to see a list of today's news stories, so that I can stay up to date on current affairs.

</user-story-example>

This list of user stories should focus on the core high-value stories. Stories are extended with acceptance criteria. Developer user stories are only allowed if the TDD is being completed for a developer facing product. For applications, implementation decisions describe technical constraints. Non-functional requirements should be expressed as measurable acceptance criteria where they relate to a specific user story. Non-functional requirements that affect multiple stories should be documented separately under Implementation Decisions.

## Story Details

### <Story Number> - <Scenario Name>

A numbered list of any relevant acceptance criteria for the story, written in a BDD (Given, When, Then) format.

#### <Story Number>.<Acceptance Criteria Number>

<story-details-example>

### 1 - App Launch

#### AC 1.1

Given a user is logged in

When the home page loads

Then the personalised list of articles is displayed.


#### 1.2

Given a user is not logged in

When the home page loads

Then a sign-in prompt is displayed.


</story-details-example>

## Implementation Decisions

A list of implementation decisions that were made or proposed, grouped by section. Sections can be omitted if they are not relevant.

### General

* The project modules that will be added or modified.
* Architectural decisions.
* Technical clarifications from the developer.

### Data Layer

* Data sources to be modified or new ones to be created.
* API contracts.

### Domain Layer

* Use cases to be modified or new ones to be created.

Do NOT include complete file paths or code snippets. They may become outdated very quickly. Class names and representations of the package structure are okay.

## Testing Strategy

Outline the testing approach for this feature, specifying what will be covered at each layer.

1. Unit tests should be added for all new classes.
2. Unit tests for existing classes should be updated to reflect the new functionality.
3. Tests should make use of a test scope DSL in order to group setup.

## Out of Scope

A numbered list describing things that are out of scope for this spec.

## Further Notes

Any further notes about the feature.

</tdd-template>
