---
description: "Create an epistemic trace from the current conversation — crystallize insights from exploratory work into a referenceable artifact. Filesystem-only, no MCP server required. Reads content_schemas from adapt.md if declared; otherwise uses the default fallback schema."
when_to_use: "After brainstorming or exploratory conversation, when insights need preserving — turning points, contradictions, open questions, formulations worth quoting back."
allowed-tools:
  - Read
  - Write
  - Glob
---

Create an epistemic trace from the current conversation.

## Steps

### Step 0 — Read content_schema from adapt.md (configurable schema, with fallback)

Read `<project_root>/adapt.md`, parse the YAML frontmatter, extract `content_schemas.trace.fields`.

- **If the block exists**: use those fields as the schema for the trace content.
- **If absent**: use the default below (fallback hardcoded — preserves backward compatibility).

**Default (fallback):** `[insights, conceptual_map, formulations, open_questions, context_forward, warnings]`

Rationale: a trace today has a fixed dialectical shape (insights / conceptual_map / formulations / open_questions / context_forward / warnings). This is *one* way to think — it preserves alternatives, holds open questions, recognizes warning patterns. Other domains may think differently (a clinician's trace, a journalist's, a reviewer's). The skill protocol stays universal; the schema belongs to the domain.

### Step 1 — adapt.md routing check

Read `adapt.md` from the project root and check the frontmatter:

- **If `adapt.md` is absent or no frontmatter:** proceed with defaults (skill enabled, write to `traces/` or localized equivalent).
- **If `skills.mhc-trace: disabled` or `artifact_types.traces: absent`:** **refuse** with routing suggestion pointing to `primary_output_location` (or to `/mhc-note` as a lighter-weight alternative). Stop.
- **If `skills.mhc-trace: redirected`:** use the redirected target path.
- **If frontmatter schema-invalid:** stop and surface the error; suggest `/mhc-onboard`.

### Step 2 — Read config

1. Read `.mhc-config.json` → extract `current_session.id` (or `.sid`), `project_name`, `folders.traces` (default `traces/` or `tracce/`).
2. If `.mhc-config.json` missing: *"No `.mhc-config.json`. Run `/mhc-onboard` first."* Stop.

### Step 3 — Assemble content (Rule 2 — surface choices)

Apply Rule 2 (Salomone): surface meaningful choices about what to capture — what insights belong in the trace, which formulations the user wants preserved verbatim, which questions remain open.

Assemble the trace content as an object with the fields declared in the active schema (Step 0):
- For default schema: `{insights, conceptual_map, formulations, open_questions, context_forward, warnings}`
- For custom schema: whatever `content_schemas.trace.fields` lists

Plus standard metadata:
- `topic`: descriptive topic of the trace
- `project`: from config
- `date`: today's ISO date
- `session_id`: current SID
- `inputs`: list of upstream materials referenced (default `[]`)

### Step 4 — Build the artifact

Frontmatter:

```yaml
---
artifact_type: epistemic_trace
topic: <topic>
project: <project_name>
date: <YYYY-MM-DD>
session_id: <current SID>
inputs: <list, or [] if none>
validated: <ISO timestamp>
validation: <approved | approved_with_edits>
---
```

Body: render the fields from the active schema, one section each.

For default schema:

```markdown
# Epistemic Trace — <topic>

## Key Insights Discovered
<bullet list — what became clear that wasn't before>

## Conceptual Map
<how do the ideas relate to each other; structure of the territory>

## Preserved Formulations
<exact phrasing the user found particularly good — quoted verbatim>

## Open Questions
<bullet list — what remains unclear, what needs further exploration>

## Context to Carry Forward
<what should a future session know to continue this work>

## Warnings / Pitfalls
<what to avoid; failure modes already encountered>
```

For custom schema: render each declared field as a `##` section.

### Step 5 — Validate (Rule 4 — Esdra)

Present the assembled markdown to the user. Ask: **Approve / Edit / Cancel**. Only save on explicit approval.

### Step 6 — Write

On approval, write to `<folders.traces>/trace_<descriptive-slug>_<YYYYMMDD>.md`. Use `Write` tool.

### Step 7 — Confirm

One-line confirmation: *"Trace saved: `<folder>/<filename>`."*

## Rules

- **Be honest about open questions.** Don't pretend certainty. If something is unresolved, mark it open — that *is* the trace's job (Apollo — surface what is assumed; Minerva — surface the trade-offs that remain).
- **Preserve user formulations verbatim.** When the user said something well, quote it exactly — don't paraphrase. Quoted formulations are first-class content in a trace.
- **Decalogo intact.** Three Principles + Seven Rules are identity.

---

*MHC-C `/mhc-trace` — epistemic trace, filesystem-only. Originally part of MHC-A framework (`MHC-A/skills/mhc-trace/SKILL.md`), extracted + retrofitted to MHC-C 2026-05-27 (removed MCP dependencies, added content_schemas Step 0 with fallback default preserved).*
