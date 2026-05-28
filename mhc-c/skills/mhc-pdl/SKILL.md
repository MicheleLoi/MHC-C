---
name: mhc-pdl
description: "Create or update a prompt development log (PDL) — document the decisions about what a prompt should generate before generating it. Filesystem-only, no MCP server required. Reads content_schemas from adapt.md if declared; otherwise uses the default fallback schema."
when_to_use: "When specifying what to generate, before producing the prompt. Each PDL entry documents an option-weighing decision: alternatives considered, choice made, rationale, downstream impact."
allowed-tools:
  - Read
  - Write
  - Glob
---

Create or update a prompt development log.

## Steps

### Step 0 — Read content_schema from adapt.md (configurable schema, with fallback)

Read `<project_root>/adapt.md`, parse the YAML frontmatter, extract `content_schemas.pdl.fields`.

- **If the block exists**: use those fields as the schema for each PDL entry.
- **If absent**: use the default below.

**Default (fallback):** `[goal, audience, constraints, draft_outline]`

Rationale: PDLs vary by domain. A research-paper PDL specifies `audience` + `theoretical_frame`. A legal-memo PDL specifies `jurisdiction` + `client_facts`. A software-spec PDL specifies `inputs` + `outputs` + `edge_cases`. The skill protocol stays universal; the domain declares fields.

### Step 1 — adapt.md routing check

- **If absent or no frontmatter:** proceed with defaults (write to `pdl/` or `decision_log/prompt_development_logs/`).
- **If `skills.mhc-pdl: disabled` or `artifact_types.pdl: absent`:** refuse with routing suggestion. Stop.
- **If `skills.mhc-pdl: redirected`:** use the redirected target path.
- **If frontmatter schema-invalid:** stop and surface; suggest `/mhc-onboard`.

### Step 2 — Read config

1. Read `.mhc-config.json` → `current_session.id`, `project_name`, `folders.pdl`.
2. If updating an existing PDL: read it first; add current SID to the session_id list.

### Step 3 — Assemble content (Rule 2 — Salomone, Rule 7 — Minerva)

Apply Rule 2 + Rule 7: surface meaningful choices, then surface the trade-offs of each.

For each PDL entry, capture:
- Fields from the active schema (Step 0). Default: `goal, audience, constraints, draft_outline`.
- Plus the option-weighing structure: **issue/need**, **options considered (≥2)**, **decision**, **rationale**, **what it affects (downstream)**.

A PDL is a record of choosing among alternatives. Don't just list the chosen path — list the alternatives the user weighed and why this one was picked over the others (Minerva — surface the cons).

### Step 4 — Build the artifact

Frontmatter:

```yaml
---
artifact_type: pdl
project: <project_name>
created: <original date if updating, else today>
last_updated: <today ISO>
status: <draft | in_progress | complete>
session_id: <SID or list>
inputs: <list>
validated: <ISO timestamp>
validation: <approved | approved_with_edits>
---
```

Body for default schema:

```markdown
# Prompt Development Log (PDL) — <topic>

## PDL-001: <entry title>

| Field | Value |
|-------|-------|
| Date | <YYYY-MM-DD> |
| Goal | <what the prompt should produce> |
| Audience | <who reads/uses the output> |
| Constraints | <hard constraints — format, length, tone, jurisdiction, etc.> |
| Draft outline | <skeleton of what the prompt covers> |

**Issue/Need:** <what triggered this entry>

**Options Considered:**
1. **Option A:** <description> — Pros: ... Cons: ...
2. **Option B:** <description> — Pros: ... Cons: ...

**Decision:** <which option>

**Rationale:** <why — surface assumptions (Apollo), surface trade-offs (Minerva)>

**What it affects:** <downstream impact — which artifacts, which decisions are now constrained>

---

## Current Prompt State

<Summary of where the prompt stands after all logged entries>
```

For custom schema: render fields per `content_schemas.pdl.fields`.

### Step 5 — Validate (Rule 4 — Esdra)

Present markdown. Ask: **Approve / Edit / Cancel**. Only save on explicit approval.

### Step 6 — Write

On approval, write to `<folders.pdl>/pdl_<descriptive-slug>_<YYYYMMDD>.md`.

### Step 7 — Confirm

One-line: *"PDL saved: `<folder>/<filename>` — entry `PDL-NNN`."*

## Rules

- **List options, not just choices.** A PDL with only "Decision: X" loses the audit value. Always list the alternatives weighed (Salomone — distinguish alternatives before asking).
- **Rationale explains WHY, not WHAT.** "Chose Option A" is the choice; "because Option B would have collapsed the audience" is the rationale.
- **Decalogo intact.**

---

*MHC-C `/mhc-pdl` — prompt development log, filesystem-only. Originally part of MHC-A framework (`MHC-A/skills/mhc-pdl/SKILL.md`), extracted + retrofitted to MHC-C 2026-05-27 (removed MCP dependencies, added content_schemas Step 0 with fallback default preserved).*
