---
name: write-a-prd
description: Create a Product Requirements Document (PRD) for a mobile application or SDK feature through structured stakeholder interviews. Use when creating a new feature, modifying existing behaviour, writing user stories, or documenting product requirements. Focus on WHAT and WHY, not HOW. If a repository is available, inspect it first to understand the existing behaviour before writing the PRD.
---

# Process

1. Understand the existing behaviour.
   * Inspect the repository if available.
   * Ask the user to connect one if the current behaviour cannot be verified.
2. Interview the stakeholder until the feature is fully understood.
   * Focus on observable behaviour.
   * Walk through the user journey screen by screen.
   * Challenge assumptions and identify missing requirements.
3. Write the PRD using the template below.
4. Save the PRD as a Markdown file.

## Domain Language

Use these terms consistently throughout the interview and PRD. Prefer them over synonyms.

### Terminology

| Term | Definition | Avoid |
|------|------------|-------|
| **Deployment** | A geographic coverage area (for example, a city or region) containing a catalogue of maps. | Zone *(except in UI)* |
| **Cluster** | A logical grouping of related maps within a deployment. | — |
| **Map** | Downloadable map data representing a navigable area. Prefer **Map** over *Venue* or *Building*. | Venue, Building |

### Journey Types

- **Outdoor Walk** — A walk outside a mapped area but within a deployment. Uses GPS positioning.
- **Indoor Walk** — A walk within a mapped area.
- **Transit Leg** — A journey leg using public transport.
- **Unmapped Leg** — A journey leg through an indoor area not covered by Waymap mapping.

### User Types

- **Sighted User**
- **Visually Impaired User**
- **Tester**

### Existing Products

- **VPS (Visual Positioning System)** — Allows a user to photograph their surroundings so Waymap can determine their location.
- **Smart Step** — Previously known as *Trace*. Tracks a user's location within a map using phone sensors.

### Data Sources

| Name | Description |
|------|-------------|
| **Map Catalogue** | The authoritative source of deployments, clusters and maps. |

### Guidance

- Use the terminology above consistently.
- Prefer extending existing concepts over introducing new ones.
- Clarify unfamiliar terminology before using it.
- If a new domain term is introduced, add it to the PRD glossary.
- Never invent a new term when an established one already exists.

### Existing Screens

Prefer extending existing screens over introducing new ones.

| Screen | Purpose |
|--------|---------|
| **Search Home** | Find places using nearby locations, favourites, recents or search results. |
| **Location Detail** | View information about a place and start navigation to that destination. |
| **Journey Planner** | Select an origin, destination and departure time, then choose a route option. |
| **Journey Summary** | Review the selected journey and start or stop navigation. |
| **Leg Detail** | View details for an individual walk or transit leg. |
| **Map HUD** | Live navigation guidance including orientation, instructions, transit information and arrival. |
| **Journey Feedback** | Rate and provide feedback on a completed journey. |

### Screen Guidance

- Identify which screen(s) are affected by the proposed behaviour.
- Walk through the user's journey screen by screen.
- Consider the different states of each affected screen, such as loading, empty, error, offline and permission denied.
- If a new screen is required, explain why an existing screen is insufficient.

## Interview Guidance

You are interviewing a Product Owner, not a developer.

Focus on:

* WHAT should happen.
* WHY it should happen.
* WHO benefits.

Avoid discussing HOW the feature should be implemented.

If the stakeholder proposes implementation details, acknowledge them then redirect the discussion back to user behaviour.

Example:

> Stakeholder: "We'll add a Bluetooth service."

Reply:

> "Let's leave implementation decisions for the design stage. What capability should Bluetooth provide for the user?"

Do not stop asking questions until the feature could be implemented without making product assumptions.

## Behaviour Exploration

For every affected screen ask:

* Why is the user here?
* How did they arrive?
* What are they trying to achieve?
* What can they see?
* What actions are available?
* Where can they go next?

Consider different states:

* First use
* Returning user
* Empty
* Loading
* Error
* Offline
* Permission denied

## User Types

Common actors include:

* Sighted user
* Visually impaired user
* Tester

Developer actors are only appropriate for SDK features.

## Technical Guidance

Ground advice in what is technically achievable.

If a requested behaviour is unlikely to be feasible, explain the constraint in plain language and help discover the underlying user need rather than debating implementation.

## PRD Guidance

The PRD describes externally observable behaviour.

Include:

* Problem statement
* Solution
* User stories
* Acceptance criteria
* Edge cases
* Out of scope
* Success criteria
* Further notes

Never include:

* Architecture
* APIs
* Classes
* Frameworks
* Database design
* Algorithms

Unless documenting the public contract of an SDK.

<prd-template>

# Problem Statement

Describe the user's problem.

# Solution

Describe the proposed behaviour from the user's perspective.

# User Stories

<user-story-example>

1. As a visually impaired user, I want to pause navigation, so that I can safely respond to interruptions without losing my journey.

</user-story-example>

# Story Details

<acceptance-criteria-example>

### 1 — Pause Navigation

#### AC 1.1

Given navigation is active

When the user selects Pause

Then guidance stops until navigation is resumed.

</acceptance-criteria-example>

# Edge Cases

|Name|Description|Handled|Expected Handling|
|---|---|:-:|---|
|Offline|Network unavailable|✓|Continue using cached data.|

# Success Criteria

Describe how the team will know the feature solved the problem.

# Out of Scope

Explicitly list excluded behaviour.

# Further Notes

Additional product decisions or assumptions.

</prd-template>