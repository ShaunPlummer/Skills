---
name: write-tickets
description: Take a conversation context, product requirements document or design document and break it down into a a series of tickets. Creating a task breakdown from a larger scope of work which is too big for a single work item.
---

# Process
 The design document, product requirements or scope of work should already be in the conversation. If it isn't, ask the user to share it and wait for it to be provided before completing the following phases. 

1. Once you have the requirements complete the 'Understand Requirements' section.
2. When you have a understanding of the requirements move to 'Determine Breakdown'.
3. Finally once the breakdown has been determined, 'Write Tickets'.
4. Review task breakdown.
5. Upload the tickets to the project issue tracker.

# 1. Understand Requirements
Understand the requirements and identify any inconsistencies or ambiguities which would impact the creating of a task breakdown. If you need clarification, create a numbered list of question and present them to the user for their response. Wait for them to replay.

# 2. Determine Breakdown
Identify key architecture decisions on which the solution is being built.

- Each phase should deliver a small, usable increment of functionality.
- Prefer thin vertical slices that cross the relevant application layers over horizontal, layer-by-layer work.
- Each phase must add user value and be verifiable.
- Keep phases small enough to integrate, review, and receive feedback quickly.
- A single user story may span multiple tickets, with each ticket delivering  delivering a distinct subset of its acceptance criteria.
- Never add non functional UI elements. 
- Each task should leave main in a 'ready to ship' state where the work completed to date could be released without buttons that do nothing or controls that are non functional.

# 3. Write Tickets
Review the provided 'EXAMPLE.md' to understand how individual tickets should be structured.

Write one file per ticket under `.scratch/<feature-slug>/issues/<NN>-<slug>.md`, numbered from `01` in dependency order (blockers first). Each file's dependencies lists the numbers/titles it depends on. Use the per-ticket 'TEMPLATE.md', one ticket per file, never a single combined file.

# 4. Review Task Breakdown

# 5. Upload


