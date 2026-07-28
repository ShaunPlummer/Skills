---
name: write-a-prd
description: Creates a Product Requirements Document (PRD) for a mobile application or SDK feature through structured stakeholder interviews. Use when creating a new feature, modifying existing behaviour, writing user stories, or documenting product requirements. Focus on WHAT and WHY, not HOW.
---

# Process

1. Interview the stakeholder until the feature is fully understood.
   * Walk through the user journey screen by screen.
   * Map navigation triggers, destinations, and back/exit state behavior for every user action.
   * Focus on observable behaviour.
   * Challenge assumptions and identify missing requirements.
2. Cross-check answers against Domain Language and Existing Screens. Flag conflicts with documented product behaviour. When current behaviour is unclear, ask the stakeholder what happens today — do not invent it.
3. Write the PRD using the template below.
4. Share the PRD as downloadable Markdown file.

## Product Context

Waymap is an accessible indoor and outdoor pedestrian navigation product for sighted and visually impaired users. Coverage is organised as Deployments containing Clusters of Maps. Journeys are made of Outdoor, Indoor, Transit, and Unmapped legs. Positioning uses GPS outdoors, Smart Step indoors, and VPS to visually relocate. Guidance is visual and spoken (TTS); spoken and on-screen content should stay aligned unless the PRD says otherwise.

Assume no repository access. Treat Domain Language and Existing Screens as the product model. When the stakeholder implies existing behaviour that is not documented here, ask what happens today rather than inferring.

## Domain Language

Use these terms consistently throughout the interview and PRD. Prefer them over synonyms.

### Terminology

| Term | Definition | Avoid |
|------|------------|-------|
| **Deployment** | A geographic coverage area (for example, a city or region) containing a catalogue of maps. | Zone *(except in UI)* |
| **Cluster** | A logical grouping of related maps within a deployment. | — |
| **Map** | Downloadable map data representing a navigable area. Prefer **Map** over *Venue* or *Building*. | Venue, Building |
| **Text to speech (TTS)** | A mechanism to announce instructions using a voice synthesiser. | — |

### Journey Leg Types

- **Outdoor** — A walk outside a mapped area but within a deployment. Uses GPS positioning.
- **Indoor** — A walk within a mapped area.
- **Transit** — A journey leg using public transport.
- **Unmapped** — A journey leg through an indoor area not covered by Waymap mapping.

### User Types

- **Sighted User**
- **Visually Impaired User**
- **Tester**

Developer actors are only appropriate for SDK features.

### Existing Products

- **VPS (Visual Positioning System)** — Allows a user to photograph their surroundings so Waymap can determine their location.
- **Smart Step** — Previously known as *Trace*. Tracks a user's location within a map using phone sensors.

### Data Sources

| Name | Description |
|------|-------------|
| **Map Catalogue** | The authoritative source of deployments, clusters and maps. |
| **Map** | A single area for which a detailed map has been created. |
| **Deployment Index** | A summary of searchable destinations within a deployment |
| **Search** | Users can search the deployment, internally this uses the deployment index |

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

Typical flow: Search Home → Location Detail → Journey Planner → Journey Summary → Map HUD → Journey Feedback. Leg Detail is a side path from Journey Summary or Map HUD.

### Screen Guidance

- Identify which screen(s) are affected by the proposed behaviour.
- Walk through the user's journey screen by screen.
- Explicitly document screen modification, layout states, and navigation transitions.
- Consider the different states of each affected screen, such as loading, empty, error, offline and permission denied.
- If a new screen is required, explain why an existing screen is insufficient.
- For SDK features, treat public interfaces/listeners as "screens." Document callback triggers, initial/pending states, error handling, and parameter validation.

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

### Story 1 — Pause Navigation

#### AC 1.1

Given navigation is active

When the user selects Pause

Then guidance stops until navigation is resumed.

</acceptance-criteria-example>

# Edge Cases

|Name|Description|Handled|Expected Handling|
|---|---|:-:|---|
|Offline|Network unavailable|✓|Continue using cached data.|

# Open Questions & Pending Decisions
| Item | Impacted Component | Stakeholder Action Needed | Target Resolution Date |
| :--- | :--- | :--- | :--- |
| Confirm max TTS announcements | Map HUD | Check with Accessibility Team | YYYY-MM-DD |

# Out of Scope

Explicitly list excluded behaviour.

# Further Notes

Additional product decisions or assumptions.

</prd-template>
