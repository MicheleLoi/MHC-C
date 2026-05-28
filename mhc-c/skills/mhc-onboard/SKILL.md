---
name: mhc-onboard
description: "First-use experience for MHC-C. Sets up a project to use the MHC framework capture skills (note, trace, modlog, pdl, output, status) as a standalone Claude Code plugin — no server, no network, no external dependencies. Handles greenfield, existing projects, and migrations from legacy MHC layouts. Generates .mhc-config.json + adapt.md + CLAUDE.md + methodology.md."
when_to_use: "When a project needs to be set up with MHC-C, or when re-running onboarding to repair / re-verify the project's adaptation file. Run once per project."
allowed-tools:
  - Read
  - Write
  - Glob
  - Grep
  - Bash
---

# /mhc-onboard — MHC-C onboarding (standalone, filesystem-only)

Set up a project to use the MHC-C Core skill set. Detect the project's current state, preserve any pre-existing convention, and apply the appropriate flow.

MHC-C runs **filesystem-only**: no MCP server, no network. The session ID is generated locally (timestamp-based), conversations are kept locally, and everything stays on the user's machine. This skill produces `.mhc-config.json` (the project state file), `adapt.md` (the project adaptation), `CLAUDE.md` (Claude's auto-loaded context), and `methodology.md` (the artifact-model reference).

## Step 1 — Detect case

Check the project root. Signals can co-occur (hybrid). Use this table to classify:

| Condition | Case |
|---|---|
| No `.mhc-config.json`, no `adapt.md`, no CLAUDE.md, no artifact folders, no work files | **Case A: Greenfield** |
| Project content present (work files, folders) but no `.mhc-config.json` and no adapt.md | **Case B: Existing project, no MHC** |
| `CLAUDE.md` contains legacy MHC markers (`router.md`, `MHC-start`, `MHC-W-main`, `agents/`) OR `.claude/settings.local.json` contains MHC-W Python-hook entries OR `.mhc-config.json` version is pre-v5 | **Case C: Legacy MHC migration** |
| `adapt.md` present with substantial prose body (>20 lines) OR custom skills in `.claude/skills/` beyond the MHC-C core set OR non-standard artifact folder layout | **Case D: Pre-adapted project** |

Case A runs inline below. Cases B / C / D run through a unified 5-phase procedure (DETECT → EXTRACT → SYNTHESIZE → VALIDATE → WRITE) documented separately; for the purposes of this skill the high-level flow is: detect what's there, preserve it where preservable, surface the proposed config + adapt.md + CLAUDE.md, validate with the user (Rule 4 — Esdra), then write.

Announce which case was detected (Rule 1 — Gabriele) before proceeding.

---

## Case A — Greenfield

### Step A.1 — Language

Ask in all four languages simultaneously:

> "Which language would you like to use? / Quale lingua vuoi usare? / Quelle langue souhaitez-vous utiliser? / Welche Sprache möchten Sie verwenden?"

Accept the answer in any form ("italiano", "Italian", "it"). Normalize to the ISO code for the config (`en` / `it` / `fr` / `de`).

### Step A.2 — Project name

Ask: *"What's the project name? (used in artifact frontmatter — defaults to folder name)"*

### Step A.2b — Project nature

Ask: *"What is the nature of this project? Choose one (default: `design_workspace`):*
- *`design_workspace` — producing drafts, documents, memos, deliverables (default shape)*
- *`governance_portfolio` — organizational governance; decisions recorded per domain; no rolling drafts*
- *`research_workspace` — academic or applied research; traces and papers are primary*
- *`code_embedded` — methodology artifacts live inside a code repository*
- *`operational_tool` — minimal; Decalogo + CLAUDE.md only, no artifact folders*
- *`hybrid` — multiple sets combined (declare each via adapt.md)*

Record the answer as `project_nature`. This value drives the canonical `adapt.md` frontmatter and the `primary_output_location` default.

### Step A.3 — Create folders

Create core folders. Present the list to the user before creating (Rule 1 — Gabriele).

Use the localized names for the selected language.

**English (default):**
```
notes/
traces/
modlogs/
pdl/
prompts/
drafts/
conversations/
```

**Italian (`language = "it"`):**
```
note/
tracce/
registro_modifiche/
pdl/
prompt/
bozze/
conversazioni/
```

**French (`language = "fr"`):**
```
notes/
traces/
journal_modifications/
pdl/
prompts/
brouillons/
conversations/
```

**German (`language = "de"`):**
```
notizen/
spuren/
aenderungsprotokolle/
pdl/
prompts/
entwuerfe/
gespraeche/
```

Use `Bash mkdir -p` for each. Verify they exist.

**For `governance_portfolio` and `research_workspace`** (or `hybrid` opting in): MHC-C standalone does **not** create the governance triad (`STATE.md`, `decision_log.md`, `synthesis/`). That value-prop — cross-session memory + decision_log surfacing + open-tension tracking + audit chain — is the **MHC-H added value** (the harness with configured authority key). MHC-C Core can be used on a governance-nature project; the user is welcome to manually create `STATE.md` + `decision_log.md` + `synthesis/` and reference them, but the MHC-C skills will not automatically read, write, or surface those files (boundary: `mhc-status` does **not** read decision_log or synthesis even if they exist locally — see `MicheleLoi/MHC-H` for the full governance-aware experience).

### Step A.4 — Generate SID and write config

MHC-C uses a **local timestamp-based SID** — no server, no API call. Format: `SID-YYYYMMDD-HHMMSS` (e.g., `SID-20260527-102449`).

Compute the SID from the current local time (ISO format, no timezone — implementation: read system clock, strftime).

Build `.mhc-config.json`:

```json
{
  "project_name": "<project name>",
  "project_nature": "<nature>",
  "language": "<en|it|fr|de>",
  "transport": "filesystem",
  "current_session": {
    "id": "<SID>",
    "sid": "<SID>",
    "started_at": "<ISO timestamp>",
    "goal": "",
    "inputs": [],
    "artifacts_produced": []
  },
  "session_history": [],
  "folders": {
    "notes": "<notes/ | note/ | ...>",
    "traces": "<traces/ | tracce/ | ...>",
    "modlogs": "<modlogs/ | registro_modifiche/ | ...>",
    "pdl": "<pdl/>",
    "prompts": "<prompts/ | prompt/>",
    "drafts": "<drafts/ | bozze/ | ...>",
    "conversations": "<conversations/ | conversazioni/ | ...>"
  },
  "onboarding": {
    "completed": true,
    "completed_at": "<ISO timestamp>",
    "case": "A",
    "mhc_version": "MHC-C v0.1",
    "transport": "filesystem"
  }
}
```

Use the `Write` tool to create the file.

Also generate `session_topology.yaml` at the project root:

```yaml
current_session: <SID>
sessions:
  <SID>:
    started_at: <ISO timestamp>
    goal: ""
    inputs: []
    artifacts_produced: []
```

### Step A.5 — Create adapt.md (canonical form with content_schemas)

Write `adapt.md` (Write tool) with the canonical template. **This is the load-bearing piece for MHC-C: the `content_schemas:` block in the frontmatter is where the user's domain declares the form of each artifact type.** The defaults below preserve backward compatibility — they are the fields hardcoded as fallback in each capture skill.

Substitute values from Steps A.1 / A.2 / A.2b / A.4.

```markdown
---
artifact_type: project_adaptation
project_name: <project name>
project_nature: <project_nature from A.2b>
language: <language code>
primary_output_location: <drafts/ | bozze/ | _org/ | ...>   # derived from project_nature
schema_version: 1
artifact_types:
  drafts: <present|absent>     # present for design_workspace; absent for governance_portfolio / operational_tool
  notes: present
  traces: present
  modlogs: present
  pdl: <present|absent>
  prompts: <present|absent>
  conversations: present
skills:
  mhc-status: enabled
  mhc-trace: enabled
  mhc-note: enabled
  mhc-pdl: <enabled|disabled>
  mhc-modlog: enabled
  mhc-output: <enabled|disabled>    # disabled when primary output folder: absent
  mhc-onboard: enabled
content_schemas:
  trace:
    fields: [insights, conceptual_map, formulations, open_questions, context_forward, warnings]
  note:
    fields: [topic, content, references]
  modlog:
    fields: [decision, rationale, files_touched, references]
  pdl:
    fields: [goal, audience, constraints, draft_outline]
  output:
    fields: [type, references, metadata]
  onboard:
    fields: [project_name, project_nature, language]
migration_provenance:
  from_case: A
  source_files_processed: []
  source_files_preserved: []
  source_files_archived: []
  source_files_discarded: []
  migration_timestamp: <ISO now>
  backup_location: null
---

# Project Adaptation — <project name>

## Language

Language: <user's choice>

## File Locations

| File | Path |
|------|------|
| Config | `.mhc-config.json` |
| Topology | `session_topology.yaml` |

## Folder Mappings

| Artifact type | Folder |
|---------------|--------|
| Notes | `<notes/ | note/ | ...>` |
| Traces | `<traces/ | tracce/ | ...>` |
| Modification logs | `<modlogs/ | registro_modifiche/ | ...>` |
| PDLs | `<pdl/>` |
| Prompts | `<prompts/ | prompt/>` |
| Drafts | `<drafts/ | bozze/ | ...>` |
| Conversations | `<conversations/ | conversazioni/ | ...>` |

## Content schemas — how to customize

The `content_schemas:` block in the frontmatter above declares, for each artifact type, the list of fields that the capture skill assembles. These are the **defaults shipped with MHC-C** (the same fields each capture skill uses as fallback when no `content_schemas` block is present).

You can **edit** the fields for any type in this project — the matching skill will read your override and use it. For example, to make `trace` clinical instead of dialectical:

```yaml
content_schemas:
  trace:
    fields: [presenting_problem, history, examination, assessment, plan]
```

The skill protocol stays universal; the schema belongs to the domain.

## Methodology

See `methodology.md` for the artifact-model reference (how traces, PDLs, drafts, modlogs, and notes relate).

---
*MHC-C — Project adaptation v0.1 — generated <today's ISO date>*
```

For non-default project natures (`governance_portfolio`, `research_workspace`, etc.): omit folder rows that don't apply on disk; set the matching `artifact_types.<key>: absent` and `skills.mhc-<skill>: disabled` in the frontmatter; set `primary_output_location` to the correct folder.

### Step A.6 — Create CLAUDE.md

Write `CLAUDE.md` (Write tool) with the standard MHC-C bootstrap:

```markdown
# <project_name>

## Session start

After opening Claude Code on this project, type `/mhc-status` to orient yourself — see SID, goal, recent session history, current adaptation. Triggered organically when you ask "cosa facciamo oggi" / "a che punto siamo" / "dove eravamo rimasti" / "stato del progetto" / "what is the state" / "where were we" / "orient me".

After orientation, read `adapt.md` if not already loaded — it is this project's behavioural-rules authority and declares the content_schemas the capture skills use.

## The Decalogo — Three Principles + Seven Rules

The Decalogo is the identity of MHC-C. Apply foundations before rules: if a foundation is violated, no rule can repair the action downstream.

### The Three Principles (epistemic foundations)

1. **Apollo** — *Assumpta patefiant*. Surface the assumptions under your action before acting. Tacit assumptions break silently when they fail; explicit ones are auditable and contestable. Every analysis declares its premises.
2. **Themis** — *Auctoritas ante actum*. Map the authority chain that legitimates the action before performing it. Do not recommend an action without naming the authority required to take it.
3. **Fides** — *Nihil sine teste*. Ground every claim in something you can cite explicitly. Source absent = claim invalid. Citation by file path, canonical reference, or verifiable record; never "I read it somewhere", never paraphrased authority, never invented citations.

### The Seven Rules (operational discipline)

1. **Gabriele** — *Nuntio et exspecto*. Announce before consequential actions, wait for confirmation.
2. **Salomone** — *Discerne et interroga*. Surface meaningful choices for the user. Distinguish alternatives before asking; never present choices as an undifferentiated block.
3. **Thot** — *Verba volant, scripta manent*. Offer to write up what was decided when substantial ground has been covered.
4. **Esdra** — *Proba omnia*. Validate artifacts with the user before saving. Treat files without a metadata header as suspect provenance — don't edit, cite, or extend them without a dedicated retrofit pass.
5. **Dioscuri** — *Alter alterum complet*. Design processes that use both human and AI strengths.
6. **Ockham** — *Entia non sunt multiplicanda praeter necessitatem*. Default: reuse what exists; addition requires justification.
7. **Minerva** — *Audi alteram partem*. When the user proposes an action or change, reason through it and surface the trade-offs (especially the cons) before executing. Never silently comply.

## Methodology

The artifact model — how traces, PDLs, drafts, modlogs, and notes relate — lives in `methodology.md`. Read it once; refer back when creating or editing any artifact. **The `SKILL.md` for each artifact type is its authoritative specification** — read it directly, don't infer format from existing files.

## Available skills

`/mhc-status`, `/mhc-trace`, `/mhc-note`, `/mhc-pdl`, `/mhc-output`, `/mhc-modlog`, `/mhc-onboard`

---
*MHC-C — Core skills for Claude Code (filesystem-only, no MCP server)*
```

### Step A.6.5 — Create methodology.md

Write `methodology.md` (Write tool) with the MHC-C artifact-model reference:

```markdown
---
artifact_type: methodology
document: MHC-C — Methodology and Artifact Model
project: <project_name>
describes: The mental model for MHC-C artifact types; the authoritative-spec status of SKILL.md files
created: <today's ISO date>
---

# MHC-C — Methodology

## The mental model

MHC-C artifacts are a **chain of reasoning that makes a project auditable back to its origin**.

The deliverable alone isn't enough. For work that matters — legal, consulting, engineering, research — what counts is that a future reader (you next month, a colleague, an auditor) can walk back from the output and reconstruct *why it looks the way it does*. Each artifact type captures a moment in that reconstruction.

## The artifact types

### Primary outputs — what you are making

**`<drafts/ | bozze/>`** — the primary outputs of the project: whatever you are producing. Domain-dependent: memos, briefs, opinions in legal work; scripts, READMEs, design docs, config files in engineering; papers and reports in research; articles and chapters in writing. Use `/mhc-output` to save one with structured metadata; never `Write` directly into the primary output folder.

### Exploration and specification — what feeds a primary output

**`<notes/ | note/>`** — quick captures, research, reference material. Use `/mhc-note`.

**`<traces/ | tracce/>`** — crystallized thinking after exploratory sessions. Use `/mhc-trace`.

**`<pdl/>`** — PDLs documenting **what** a draft should contain, before generating it. Use `/mhc-pdl`.

**`<prompts/ | prompt/>`** — finished AI prompts derived from a PDL.

### Revision — what records changes to a primary output

**`<modlogs/ | registro_modifiche/>`** — revision decision logs for edits to drafts or prompts. Use `/mhc-modlog`. Drafts do not change silently.

### Auto-managed

**`<conversations/ | conversazioni/>`** — exported session conversations.

## How the artifact types relate

- A *trace* explores a question; it may lead to a *PDL* that specifies what to generate; the PDL produces a *prompt*; the prompt yields a *draft* (primary output).
- A *note* is lightweight — reference material, an idea, a discovery. It may become any of the above, or stay as-is.
- A *modlog* is always retrospective: it documents what changed in a draft and why. Write the modlog *before* editing the draft.

Not every draft has a trace and a PDL upstream. Ad-hoc drafts are fine. But the chain exists when you need it, and making it explicit in the `references:` field keeps the provenance auditable.

## The single question this model answers

*Can a cold reader, opening the project six months from now, reconstruct why the deliverable looks the way it does?*

If yes, the methodology is working. If no, something in the chain is missing or broken — that is where to look.

## Skills are the specifications

The `SKILL.md` for each artifact type is the **authoritative specification** for that type. Before creating or editing a trace, PDL, note, draft, modlog, or prompt, read the relevant `SKILL.md` directly. Do not infer the format from existing files — they may be incorrect.

## Content schemas — universality + configurability

Each capture skill (`mhc-note`, `mhc-trace`, `mhc-modlog`, `mhc-pdl`, `mhc-output`) reads `content_schemas.<type>.fields` from `adapt.md` frontmatter. If declared, the skill uses those fields. If absent, the skill falls back to its hardcoded default (preserving backward compatibility).

This pattern lets you reshape the **content** of each artifact type per project without forking the skill itself. The skill is universal — the protocol is invariant (read config, surface choices, assemble typed object, validate with user, write). The schema is domain-owned — the fields belong to the way of thinking of *this* project's work.

To customize: edit the `content_schemas:` block in this project's `adapt.md`. To roll back to defaults: delete the block (the fallback hardcoded in each skill takes over).

## Boundaries of MHC-C (vs MHC-H)

MHC-C is the **standalone Core**: capture skills (note, trace, modlog, pdl, output, onboard) + light orientation (status) + filesystem-only operation. No server, no network, no cross-session memory beyond what `.mhc-config.json` records locally, no audit chain, no decision_log surfacing, no open-tension tracking.

For the **governance value-prop** — cross-session memory, append-only audit chain, surfacing of open contradictions, drift detection against immutable sources, encrypted custodial server — see `MicheleLoi/MHC-H` (in development). MHC-H integrates with MHC-C: same skills, same Decalogo, plus the harness that records every artifact to a configured-authority server.

If a project needs **governance triad** (STATE.md + decision_log + synthesis), it can scaffold those files manually and use MHC-C's capture skills to write into them. But MHC-C's `mhc-status` does **not** read or surface decision_log / synthesis content — that surfacing is the unlock of MHC-H.
```

### Step A.7 — Safe-overwrite backup (optional)

Ask: *"Should I keep a backup of every file before I change it? (uses local git — backups stay on your computer, no network)"*

On yes:
1. `Bash git -C "<project root>" init`
2. Create `.gitignore` in the project root (include `.mhc-config.json` and any sensitive folders)
3. `Bash git -C "<project root>" config user.name "Project User"`
4. `Bash git -C "<project root>" config user.email "user@local"`
5. `Bash git -C "<project root>" add -A`
6. `Bash git -C "<project root>" commit -m "Initial commit: MHC-C onboarding"`

On no: add `"git_repo": false` to config.

Skip silently if `.git` already exists at the project root.

### Step A.8 — Confirm

Tell the user:
- Folders created
- SID assigned (`<SID>`)
- `.mhc-config.json` + `adapt.md` + `CLAUDE.md` + `methodology.md` written
- 7 MHC-C skills enabled by default — invoke any of them with `/mhc-<skill>`
- To customize artifact content: edit `content_schemas:` in `adapt.md`
- To orient at next session: type `/mhc-status`, or just say "cosa facciamo oggi" / "where were we"

Then add this reminder:

> **Tip — browse your artifacts:** [Obsidian](https://obsidian.md) is a free markdown viewer and knowledge base app. All MHC-C artifacts are plain markdown — open your project folder in Obsidian and every note, trace, and conversation becomes navigable and linkable. Download free at obsidian.md.

---

## Cases B / C / D — unified procedure (5-phase)

These cases (and any hybrid combination) run a 5-phase unified procedure:

1. **DETECT** — build a detection state YAML block (scan `.mhc-config.json`, `adapt.md`, `CLAUDE.md`, `.claude/skills/`, artifact folders, legacy markers). Surface the YAML verbatim. Classify case(s). Infer `project_nature` with rationale. Ask the user to confirm or edit.

2. **EXTRACT** — per source, preserve usable content / mark discardable. Build the `extracted` structure.

3. **SYNTHESIZE** — build the canonical `adapt.md` from preserved content + defaults. Populate frontmatter from detection + extraction + inferred project_nature. Populate body from preserved prose sections. **For Cases B / C / D where `content_schemas:` is not already declared, write the defaults block** (same as Step A.5) so the capture skills work out of the box.

4. **VALIDATE** — present (a) proposed `adapt.md`, (b) source→destination diff, (c) discarded list, (d) **Approve / Edit / Cancel** prompt. **Rule 4 — Esdra.** Do not write without explicit approval.

5. **WRITE** — after approval, in strict order:
   - Create `.adapt-migration-backup-<YYYY-MM-DD>/` (append `-N` if exists)
   - Move archivable sources into the backup
   - Write `adapt.md`
   - Update `.mhc-config.json` onboarding block (`case`, `project_nature`, `migrated_at`, `backup_location`)
   - Regenerate `CLAUDE.md` from Step A.6 template if legacy / missing
   - Write `methodology.md` if absent (Step A.6.5)
   - Consistency-check the written adapt.md (YAML parses, paths exist, `project_nature` valid)
   - Report to user: cases, files archived, skills enabled, primary output location.

### Case-specific notes inside the unified flow

- **Case B (no prior MHC):** typically no backup needed (no MHC-era sources to archive). Existing project content (drafts/, README.md, folder layout) still drives project_nature inference. `migration_provenance.from_case: B`, `backup_location: null` if nothing archived.

- **Case C (legacy MHC):** always archives the legacy `CLAUDE.md`, the legacy `.claude/settings.local.json` (or relevant excerpt), and any pre-v5 `.mhc-config.json` fields. Clean MHC-W Python hooks out of `settings.local.json`. Regenerate `CLAUDE.md` from the current template. `migration_provenance.from_case: C`.

- **Case D (pre-adapted):** preserve `.claude/skills/<custom_skill>/SKILL.md` directories untouched — do NOT archive, do NOT overwrite. List them in the canonical `adapt.md` body under `## Custom skills`. Archive only the old `adapt.md` (after extracting its prose into the new canonical form). `migration_provenance.from_case: D`.

- **Hybrid:** run extraction for every applicable source; produce a single canonical `adapt.md`; `migration_provenance.from_case: "B+D"` (or similar concatenation).

### Failure modes

- **User declines at VALIDATE:** leave project untouched. No backup created. No changes.
- **WRITE fails partway after archive:** report to user. Backup folder contains archived sources — recovery is manual. Do not attempt auto-rollback.
- **Consistency check after WRITE reports problems:** report to user; suggest `/mhc-onboard` re-run or manual correction.

---

## After onboarding

Hand back to normal session work. Remind the user once more: **type `/mhc-status` at the beginning of every future Claude Code session** (or just say "cosa facciamo oggi" — the skill is triggered organically by orientation prompts).

---

*MHC-C `/mhc-onboard` — project bootstrap, filesystem-only. Originally part of MHC-A framework (`MHC-A/skills/mhc-onboard/SKILL.md`), extracted + retrofitted to MHC-C 2026-05-27 (removed MCP `mhc_start_session` dependency in favour of local timestamp-based SID; added `content_schemas:` defaults block to generated adapt.md per founder directive SID-20260527-102449; rebranded MHC-A → MHC-C in generated CLAUDE.md and methodology.md templates; Decalogo intact).*
