---
name: write-tickets
description: Take a conversation context, product requirements document or design document and break it down into a a series of tickets. Creating a task breakdown from a larger scope of work which is too big for a single work item.
---

# Process
 The design document, product requirements or scope of work should already be in the conversation. If it isn't, ask the user to share it and wait for it to be provided before completing the following phases. 

1. Once you have the requirements complete the 'Understand Requirements' section.
2. When you have a understanding of the requirements move to 'Determine Breakdown'.
3. Get the breakdown list approved in 'Breakdown Checkpoint'.
4. Finally once the breakdown has been approved, 'Write Tickets'.
5. Complete 'Review Task Breakdown'.
6. Breakdown sign off
7. Upload the tickets to the project issue tracker.

# 1. Understand Requirements
Understand the requirements and identify any inconsistencies or ambiguities which would impact the creating of a task breakdown. If you need clarification, create a numbered list of question and present them to the user for their response. Wait for them to replay.

Record every finding which you do not turn into a ticket in `.scratch/<feature-slug>/open-questions.md`. This includes contradictions in the source documents and findings which the user puts out of scope. For each finding, state where you found it and why it is not in a ticket.

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
- Enabling work, such as dependencies, assets and documentation, goes in the first ticket which needs it. Never give enabling work its own ticket.
- A setting ships in the ticket which reads its value. Do not add a setting without its control, or a control without its setting.
- Validate an assumption with verification steps in the ticket whose design it de-risks. Do not write a spike ticket.
- For a ticket which is clearly larger than its neighbours, state the seam you considered splitting it on and why the seam does not open. If you cannot name a seam, the ticket is too large.

# 3. Breakdown Checkpoint
Present the breakdown as a numbered list. Give one line per ticket and the ticket numbers it depends on. Do not write any ticket bodies yet.

Ask the user to approve the list. Wait for them to approve.

A change to the list at this point costs nothing. After the bodies exist, the same change costs a renumber of every file and every reference to it.

# 4. Write Tickets
Review the provided 'EXAMPLE.md' to understand how individual tickets should be structured.

Write one file per ticket under `.scratch/<feature-slug>/issues/<NN>-<slug>.md`, numbered from `01` in dependency order (blockers first). Each file's dependencies lists the numbers/titles it depends on. Use the per-ticket 'TEMPLATE.md', one ticket per file, never a single combined file.

Each ticket should include the user story and acceptance criteria is implemented. The numbering should be retained to provide traceability. 

The ticket file is the issue body. Write only Markdown which the issue tracker accepts. Do not use HTML tags or HTML entities.

# 5. Review Task Breakdown

- Compare the written tickets against the guidance in step 2 and make any improvement.
- Trace each in-scope requirement to a ticket containing its user story and acceptance criterion. 
- Check implementation decisions, testing requirements are included in the relevant ticket.
- Read `open-questions.md` again. Move any finding which the breakdown now covers into its ticket.


# 6. Ticket Sign off
As the user to review the tickets and approve them. Wait for them to approve.  

# 7. Upload
Once the user has approved the tickets create the issues on GitHub. se the project’s issue types, labels, and field mappings where applicable.

## Pre-flight
1. List the labels, milestones and issue types which the project already has.
2. Present the labels and milestones you intend to create.
3. Ask the user to confirm. Wait for them to confirm.

## Create the issues
Upload each ticket file as the issue body with `gh issue create --body-file`. Use the text under `## Title` as the issue title.

Write `.scratch/<feature-slug>/issues/manifest.md` as you go. Record the ticket number, the file name and the issue number for each issue you create. The `gh issue create` calls are not idempotent. The manifest lets you continue after a failure instead of starting again.

## Rewrite the dependency links
When all issues exist, replace the ticket numbers in each uploaded body with the issue numbers from the manifest. Never assume that ticket `NN` is issue `#NN`.

