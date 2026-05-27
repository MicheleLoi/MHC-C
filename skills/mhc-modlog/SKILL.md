---
description: "Create or update a modification log — track intellectual decisions during revision of any existing artifact (document, script, spec, prompt, config, skill file). Filesystem-only, no MCP server required. Reads content_schemas from adapt.md if declared; otherwise uses the default fallback schema."
when_to_use: "When revising any existing artifact and the changes carry intellectual decisions (not typos, not pure reformatting). Key question: is this a revision of something that already exists, or a first draft? If revision with intellectual decisions, offer modlog."
allowed-tools:
  - Read
  - Write
  - Glob
---

Create or update a modification log.

## Steps

### Step 0 — Read content_schema from adapt.md (configurable schema, with fallback)

Read `<project_root>/adapt.md`, parse the YAML frontmatter, extract `content_schemas.modlog.fields`.

- **If the block exists**: use those fields as the schema for each modlog entry.
- **If absent**: use the default below (fallback hardcoded).

**Default (fallback):** `[decision, rationale, files_touched, references]`

Rationale: the form of a modlog entry varies by domain. A code modlog wants `files_touched` + commit refs; a paper modlog wants `section_affected` + reasoning chain; a contract modlog wants `clause_id` + risk_impact. The skill is universal; the schema lives in `adapt.md` for the domain.

### Step 1 — adapt.md routing check

- **If `adapt.md` absent or no frontmatter:** proceed with defaults (write to `modlogs/` or `registro_modifiche/`).
- **If `skills.mhc-modlog: disabled` or `artifact_types.modlogs: absent`:** refuse with routing suggestion. Stop.
- **If `skills.mhc-modlog: redirected`:** use the redirected target path.
- **If frontmatter schema-invalid:** stop and surface; suggest `/mhc-onboard`.

### Step 2 — Read config

1. Read `.mhc-config.json` → `current_session.id`, `project_name`, `folders.modlogs`.
2. If existing modlog being updated, **read it first** (Read tool). Convert `session_id` to a list including both the original SID and the current one. Note which entries are from which session.

### Step 3 — Assemble content (Rule 2 — Salomone)

Surface choices about what changed and why. For each modification entry, capture:
- Fields from the active schema (Step 0). Default: `decision, rationale, files_touched, references`.
- Plus standard entry metadata: `entry_id` (next sequential `MOD-NNN`), `date`, `type` (categorical — see categories below).

**Type categories (suggested, free-text label allowed):**
- *For text documents (papers, drafts, specs, prompts):* Conceptual Restructure, Epistemic Calibration, Scope Adjustment, Strategic Reorientation, Clarification, Evidence Update
- *For software artifacts (scripts, configs, skill files, templates):* Bug Fix, Logic Correction, Behavioral Change, Interface Change, Scope Trim

### Step 4 — Build the artifact

Frontmatter:

```yaml
---
artifact_type: modlog
document: <what is being revised>
output_file: <path to the file being modified>
project: <project_name>
created: <original creation date if updating, else today>
last_updated: <today ISO>
session_id: <SID or list of SIDs if updating>
inputs: <list>
validated: <ISO timestamp>
validation: <approved | approved_with_edits>
---
```

Body: a numbered list of `MOD-NNN` entries. Each entry renders the fields from the active schema as a small block.

For default schema `[decision, rationale, files_touched, references]`:

```markdown
# Modification Log — <document>

## MOD-001
| Field | Value |
|-------|-------|
| Date | <YYYY-MM-DD> |
| Type | <category> |

**Decision:** <what was decided>

**Rationale:** <why — the intellectual decision, not the mechanical edit>

**Files touched:** <list>

**References:** <list of related artifacts: traces, PDLs, other modlogs>

---
```

For multi-session modlogs, label entries with their session SID if it differs from creation.

### Step 5 — Validate (Rule 4 — Esdra)

Present the assembled markdown to the user. Ask: **Approve / Edit / Cancel**. Only save on explicit approval.

### Step 6 — Write

On approval, write to `<folders.modlogs>/modlog_<descriptive-slug>_<YYYYMMDD>.md`. For multi-session modlogs, **keep the original creation date in the filename**, not the current date.

### Step 7 — Confirm

One-line: *"Modlog saved: `<folder>/<filename>` — entry `MOD-NNN`."*

## Recognizable patterns

Offer this skill when you see an existing artifact being revised with intellectual decisions behind the changes:

- Document or draft reworked across rounds (reordering sections, changing argument, adjusting claims)
- Spec updated to add, remove, or change a mechanism
- Script or config edited with intent changes (not a rewrite from scratch)
- Prompt template adjusted after testing revealed drift or failure
- Skill file updated with changed instructions, triggers, or steps

Do **not** offer for: first-draft creation, pure reformatting with no intellectual decision, trivial typo fixes.

## Rules

- **Modlog before edit.** Write the modlog *before* changing the artifact. The modlog captures the decision; the edit implements it. Inverting the order risks losing the decision rationale.
- **Decalogo intact.** Apollo (surface the assumption that drove this change), Minerva (state the cons of the chosen direction), Esdra (validate before saving).

---

*MHC-C `/mhc-modlog` — modification log, filesystem-only. Originally part of MHC-A framework (`MHC-A/skills/mhc-modlog/SKILL.md`), extracted + retrofitted to MHC-C 2026-05-27 (removed MCP dependencies, added content_schemas Step 0 with fallback default preserved).*
