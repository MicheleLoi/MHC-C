---
description: "Quick capture — save an insight, decision, or reference as a lightweight artifact. Filesystem-only, no MCP server required. Reads content_schemas from adapt.md if declared; otherwise uses the default fallback schema."
when_to_use: "When something needs capturing quickly without the overhead of a full trace or modlog — an observation, a reusable snippet, a decision made in passing, a reference to keep handy, anything that does not fit elsewhere."
allowed-tools:
  - Read
  - Write
  - Glob
  - Bash
---

Create a quick-capture note.

## Steps

### Step 0 — Read content_schema from adapt.md (configurable schema, with fallback)

Read `<project_root>/adapt.md`, parse the YAML frontmatter, extract `content_schemas.note.fields`.

- **If the block exists in the project's `adapt.md` frontmatter**: use those fields as the schema for the note content.
- **If `content_schemas` block does not exist or `note` is missing**: use the default below (fallback hardcoded — preserves backward compatibility).

**Default (fallback):** `[topic, content, references]`

Rationale: the content-schema is the *way of thinking* of a domain. The skill protocol is universal (read config, assemble typed object, validate, write). The schema lives in `adapt.md` because the form of the artifact belongs to the domain that uses it, not to the skill. The fallback preserves a working default so a project without `content_schemas:` declared still works.

### Step 1 — adapt.md routing check (artifact skill contract, schema v1)

Read `adapt.md` from the project root and check its YAML frontmatter:

- **If `adapt.md` is absent or has no frontmatter:** proceed with defaults (this skill enabled, writes to `notes/` or the localized equivalent).
- **If `skills.mhc-note: disabled` or `artifact_types.notes: absent`:** **refuse** with routing suggestion pointing to `primary_output_location`. Stop.
- **If `skills.mhc-note: redirected`:** use the redirected target path as the write location.
- **If frontmatter schema-invalid:** stop and surface the error; suggest `/mhc-onboard`.

### Step 2 — Read config

1. Read `.mhc-config.json` (Read tool). Extract:
   - `current_session.id` (or `current_session.sid`) → current SID
   - `project_name` → project name
   - `folders.notes` → write folder path (default: `notes/` or `note/` for Italian)
2. If `.mhc-config.json` does not exist: surface this — *"No `.mhc-config.json` in this project. Run `/mhc-onboard` first."* Stop.

### Step 3 — Assemble content

Apply Rule 2 (Salomone — surface meaningful choices): ask the user briefly what to capture. Suggest topic / brief content / any references the user wants linked.

Assemble the note content as an object with the fields declared in the active schema (Step 0):
- For default schema: `{topic, content, references}`
- For project-declared custom schema: whatever fields `content_schemas.note.fields` lists

Plus the standard metadata for every note (these are NOT in the content_schema; they are required for every artifact):
- `note_id`: next sequential `NOTE_NNN` (scan existing notes folder + increment) or descriptive ID
- `title`: brief title (one line)
- `project`: from config
- `created`: today's ISO date
- `session_id`: current SID from config
- `inputs`: list (default `[]`)

### Step 4 — Build the artifact (frontmatter + body)

Assemble the markdown file content. Frontmatter block:

```yaml
---
artifact_type: note
note_id: <NOTE_NNN or descriptive id>
title: <title>
project: <project_name>
created: <today's ISO date>
session_id: <current SID>
inputs: <list, or [] if none>
validated: <ISO timestamp when validation happens>
validation: <approved | approved_with_edits>
---
```

Body: render the fields declared in the active schema (Step 0), one section each:
- For default `[topic, content, references]`:
  ```markdown
  # NOTE_NNN — <title>

  ## Topic
  <topic>

  ## Content
  <content>

  ## References
  <list of references, or empty if none>
  ```
- For custom schema: render each field as a `## <Field name capitalized>` section with the user-provided content.

### Step 5 — Validate (Rule 4 — Esdra)

Present the assembled markdown to the user. Ask: **Approve / Edit / Cancel**.

- Only save on explicit approval. Set `validation: approved` in the frontmatter (or `validation: approved_with_edits` if the user edited before approving).
- Never save without approval.

### Step 6 — Write

On approval, write to `<folders.notes>/note_<descriptive-slug>_<YYYYMMDD>.md`:
- Use `Bash mkdir -p` only if the folder does not exist (allowed-tools includes Bash for this).
- Use `Write` tool to create the file.

### Step 7 — Confirm

One-line confirmation: *"Note saved: `<folder>/<filename>` — note_id `NOTE_NNN`."*

## Rules

- **Decalogo intact.** Apollo / Themis / Fides + Gabriele / Salomone / Thot / Esdra / Dioscuri / Ockham / Minerva are identity. This skill cites them by name (Esdra for validation, Salomone for surfacing choices).
- **No fabrication.** Only capture what the user states. If references are unclear, ask.
- **No silent edits.** If the user invokes `/mhc-note` to update an existing note, refuse — point them at `/mhc-modlog` instead.

---

*MHC-C `/mhc-note` — quick capture, filesystem-only. Originally part of MHC-A framework (`MHC-A/skills/mhc-note/SKILL.md`), extracted + retrofitted to MHC-C 2026-05-27 (removed MCP dependencies, added content_schemas Step 0 with fallback default preserved).*
