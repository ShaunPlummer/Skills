---
name: obsidian-vault
description: Search, create, and organize notes in an Obsidian vault. Use when the user mentions Obsidian, the vault, work notes, meeting minutes, weekly notes, folder notes, proposals, attachments, archive, or wants to find/create/edit markdown notes outside a code repo.
disable-model-invocation: true
---

# Obsidian Vault

Agent guidance for working in an Obsidian vault. Conventions live in the vault; this skill covers how the agent must behave.

## Vault

- Path: `C:\Users\shaun\Obsidian\Work-Notes`
- Conventions SSOT: `Obsidian.md` at the vault root
- Read `Obsidian.md` before creating or reorganizing notes. Follow those rules. Do not invent a different layout, naming scheme, link style, or attachment layout.

## Agent rules

- Prefer absolute paths under the vault. Do not assume the workspace root is the vault unless it already is.
- Use Glob/Grep against the vault path (not the current workspace) unless the workspace *is* the vault.
- For placement, also open the nearest folder hub note when one exists.
- Writing standard, naming, links, attachments, templates, and folder structure: follow the vault conventions file only. Do not restate or invent variants in this skill.
- After creating or moving a note, add or update its descriptive link in the nearest folder note.
- If you add, rename, or remove a top-level or product-area folder that appears in the conventions Structure tree, update that tree in the same change.
- Do not edit `.obsidian`, delete notes, rename folders, or perform bulk reorganizations unless explicitly requested.
- In Ask mode: search and read only. Do not create, edit, move, or delete vault files.

## Workflows

### Search

```text
Glob: **/*keyword*.md  under <vault-path>
Grep: content search under the same root, *.md
```

Start from area folder hubs when the topic is clear. Use Glob/Grep when the location is unknown.

### Create a note

1. Read the vault conventions file.
2. Choose the correct folder from vault conventions; create a folder note if you create a new folder.
3. Name the file per Naming in the conventions file (e.g. Title Case).
4. For meetings or weeklies, use the matching template and path patterns from the conventions file.
5. Write the note body per the writing standard in the conventions file.
6. Link with Markdown vault-relative paths.
7. Store files under `attachments/` per the conventions file (no vault-root dumps, no overwrites).
8. Update the nearest folder hub with a descriptive link.
9. Update the Structure tree in the conventions file only if this change adds/renames/removes a documented area folder.

### Edit / organize

- Prefer updating the folder hub when summarizing or indexing a topic.
- Keep proposal naming until agreed; then rename as conventions require and fix links.
- Do not scatter attachments at vault root or use opaque paste names when a clear name exists.
- Prefer archive locations from the conventions file over delete when retiring material (only when the user asks to archive or the task clearly requires it).

### Meetings and weeklies

- Paths and date/week rules: conventions file → Naming.
- Use the templates named in the conventions file.
- Keep template frontmatter; fill the body per the writing standard.
