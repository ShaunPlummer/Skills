---
name: write-tickets
description: Take a conversation context, product requirements document or design document and break it down into a a series of tickets. Creating a task breakdown from a larger scope of work which is too big for a single work item.
---

# Process
 The design document, product requirements or scope of work should already be in the conversation. If it isn't, ask the user to share it and wait for it to be provided before completing the following phases. 

1. Once you have the requirements complete the 'Understand Requirements' section.
2. When you have a understanding of the requirements move to 'Determine Breakdown'.
3. Finally once the breakdown has been determined, 'Write Tickets'.
4. Complete 'Review Task Breakdown'.
5. Breakdown sign off
6. Upload the tickets to the project issue tracker.

# 1. Understand Requirements
Understand the requirements and identify any inconsistencies or ambiguities which would impact the creating of a task breakdown. If you need clarification, create a numbered list of question and present them to the user for their response. Wait for them to replay.

# 2. Determine Breakdown
The next step to identify how to break the development effort down into a series of tasks which can be worked on individually. This breakdown should focus on vertical slices, not horizontal layers. Each task should incrementally deliver functionality while keeping the main branch in a always ready to ship state. This means do not add non functional UI elements or placeholder function to be implemented in future work. 

Start by identifying key architecture decisions on which the solution is being built.

## Break Down Rules
- Each ticket should deliver a small, usable increment of functionality.
- Prefer thin vertical slices that cross the relevant application layers over horizontal, layer-by-layer work.
- Each ticket must add user value and be verifiable.
- Keep tickets small enough to integrate, review, and receive feedback quickly.
- A single user story may span multiple tickets, with each ticket delivering  delivering a distinct subset of its acceptance criteria.
- Never add non functional UI elements.
- Refactoring tasks are allowed. They must not change the app functionality or user facing behaviour. They can only change implementaion details and how functionality is implemented.

# 3. Write Tickets
Review the provided 'EXAMPLE.md' to understand how individual tickets should be structured.

Write one file per ticket under `.scratch/<feature-slug>/issues/<NN>-<slug>.md`, numbered from `01` in dependency order (blockers first). Each file's dependencies lists the numbers/titles it depends on. Use the per-ticket 'TEMPLATE.md', one ticket per file, never a single combined file.

Each ticket should include the user story and acceptance criteria is implemented. The numbering should be retained to provide traceability. 

# 4. Review Task Breakdown

- Compare the written tickets against the guidance in step 2 and make any improvement.
- Trace each in-scope requirement to a ticket containing its user story and acceptance criterion. 
- Check implementation decisions, testing requirements are included in the relevant ticket.


# 5. Ticket Sign off
As the user to review the tickets and approve them. Wait for them to approve.  

# 6. Upload
Once the user has approved the tickets create the issues on GitHub. se the project’s issue types, labels, and field mappings where applicable.

