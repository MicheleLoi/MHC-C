---
name: mhc-output
description: "Save a primary output (deliverable) with structured frontmatter and optional references to upstream artifacts. Primary outputs are the products of a project — memos, briefs, scripts, papers, articles, whatever the project is making. Every other artifact type (notes, traces, PDLs, modlogs) supports, specifies, or revises them. Filesystem-only, no MCP server required."
when_to_use: "When you and the user have produced something meant to stand as a primary output. Use this to save the output into the project's primary-output folder with frontmatter and provenance references, not a plain Write."
allowed-tools:
  - Read
  - Write
  - Bash
---

# /mhc-output — Save a primary output with structured metadata

A **draft** in MHC-C terms is a primary output — *whatever you are producing*. That is domain-dependent:

- Legal work: memos, briefs, contracts, opinions, pleadings.
- Engineering: Python scripts, READMEs, config files, design docs, API specs.
- Research: papers, reports, proposals, literature reviews.
- Writing: articles, chapters, pitches, essays.

The shared property: *it is the thing this project is trying to make.*

Everything else in the artifact chain supports the primary output:

- **Traces** explore; they may lead to a draft.
- **PDLs (prompt development logs)** specify what a draft should contain before generating it.
- **Prompts** are finished specifications derived from a PDL.
- **Notes** capture lightweight material that may feed any of the above.
- **Modlogs** record revisions *to* a draft after the fact.

This skill saves a draft with frontmatter that makes those relationships explicit and machine-readable. Do not use a plain `Write` to create a primary output — it loses provenance.

---

## Steps

### Step 0 — Read content_schema from adapt.md (configurable schema, with fallback)

Read `<project_root>/adapt.md`, parse the YAML frontmatter, extract `content_schemas.output.fields`.

- **If the block exists**: use those fields as the metadata schema for the output frontmatter.
- **If absent**: use the default below.

**Default (fallback):** `[type, references, metadata]`

Rationale: outputs vary wildly in form (memo, script, paper) but always have a *kind* (`type`), often a *provenance chain* (`references`), and sometimes domain-specific *metadata* (jurisdiction, audience, license, etc.). The default captures the universal triad; domains can extend via `adapt.md`.

### Step 1 — adapt.md routing check (artifact skill contract, schema v1)

Read `adapt.md` frontmatter:

- **If absent or no frontmatter:** proceed with defaults (this skill enabled; writes to `drafts/` or `bozze/`).
- **If `skills.mhc-output: disabled`:** **refuse**. *"This project's `adapt.md` has disabled `/mhc-output` — primary outputs go to `<primary_output_location>`. Capture the work there instead, or use a different skill if appropriate."* Stop.
- **If `artifact_types.drafts: absent`:** refuse with the same routing pointer.
- **If `skills.mhc-output: redirected`:** use the redirected target path (declared in `routing_rules` or `.mhc-config.json` `folders.drafts`).
- **If frontmatter parses but validation fails:** stop, surface the error, suggest `/mhc-onboard`.

### Step 2 — Locate the content

The content is normally produced in conversation just before this skill is invoked. If the user invokes `/mhc-output` cold, ask: *"What content should go in the draft? (paste it, or describe what to include)"*

### Step 3 — Collect the metadata

Ask for each (accept concise answers):

- **Title:** short, describes the deliverable.
- **Draft type (`type` field in schema):** a short free-text label. Whatever fits the domain; one word when possible, hyphenate if needed. Examples:
  - Legal: `memo`, `brief`, `contract`, `opinion`, `pleading`
  - Engineering: `python-script`, `readme`, `config`, `claude-md`, `design-doc`, `api-spec`
  - Research: `paper`, `report`, `proposal`, `literature-review`
  - Writing: `article`, `chapter`, `pitch`, `essay`
  - Any other label that conveys meaning in the project's domain.
- **Upstream references (optional):** relative paths to the artifacts this draft flows from. Examples:
  - a PDL: `decision_log/prompt_development_logs/pdl_memo_<topic>_<date>.md`
  - a trace: `traces/trace_<topic>_<date>.md`
  - a prompt: `prompts/prompt_<topic>.md`
  - a note: `notes/note_<topic>_<date>.md`
  - If no upstream artifact exists, leave the field empty.

If the conversation already shows a PDL or trace that clearly led to this draft, propose it as a default and confirm with the user.

### Step 4 — Generate the filename

Slug format: `draft_<short-slug>_<YYYYMMDD>.md`. Example: `draft_memo_art13_<topic>_20260527.md`. Ask the user to confirm or edit.

### Step 5 — Assemble the file

Frontmatter (default schema):

```yaml
---
artifact_type: draft
type: <free-text label — see Step 3 examples>
title: <title>
project: <from .mhc-config.json project_name>
created: <today's ISO date>
session_id: <current SID>
references: <list, or [] if none>
metadata: <optional domain-specific map, or {} if none>
---
```

Followed by the content provided in Step 2.

If `adapt.md` declares a custom `content_schemas.output.fields`, render those fields in the frontmatter instead.

### Step 6 — Write to the primary-output folder

Use the path from `.mhc-config.json` `folders.drafts` (default: `drafts/` or `bozze/`). If the folder does not exist, create it with `Bash mkdir -p`. Use the `Write` tool to create the file.

### Step 7 — Confirm to the user

One-line confirmation: *"Draft saved: `drafts/<filename>` — referenced from <N> upstream artifact(s)."* (Or *"no upstream references"* if none.)

### Step 8 — Remind about the next link in the chain

If the draft is likely to be revised later, mention once: *"When you revise this, use `/mhc-modlog` so the revision decisions are recorded alongside the draft."*

---

## Rules

- **Never write to the primary-output folder except through this skill.** A plain `Write` into `drafts/` loses the frontmatter, the session link, and the upstream provenance. If you catch yourself about to `Write` into `drafts/`, invoke this skill instead.
- **Do not fabricate references.** Only list upstream artifacts the user names, or ones visible in the current conversation where the link is unambiguous.
- **Do not silently revise a draft.** If the user asks to edit an existing draft, use `/mhc-modlog` to capture the revision rationale before changing the file.
- **Decalogo intact.** Apollo (surface what the output assumes), Themis (name the authority that legitimates the output), Esdra (validate before saving).

---

*MHC-C `/mhc-output` — primary output capture, filesystem-only. Originally part of MHC-A framework (`MHC-A/skills/mhc-output/SKILL.md`), extracted + retrofitted to MHC-C 2026-05-27 (added content_schemas Step 0 with fallback default preserved; reframing MHC-A → MHC-C wording; no MCP changes — was already filesystem-only).*
