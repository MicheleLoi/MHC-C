---
description: "Show current session state and orient yourself in the project. Reads .mhc-config.json + session_topology.yaml + adapt.md frontmatter to surface: current SID, goal, recent session history, skills enabled, drift between adapt.md and CLAUDE.md. Triggered organically when the user asks 'cosa facciamo oggi', 'a che punto siamo', 'dove eravamo rimasti', 'stato del progetto', 'orientami', 'riassumi', 'come va', 'what is the state', 'where were we', 'orient me', 'recap'."
when_to_use: "When the user opens a session with contextual orientation prompts (cosa facciamo oggi, a che punto siamo, dove eravamo rimasti, stato del progetto, orientami, riassumi, come va, what is the state, where were we, orient me, recap), or any time they explicitly want to know the current MHC session state. Filesystem read-only — no MCP server, no network, no sensitive data leaves the machine."
allowed-tools:
  - Read
  - Glob
  - Write
  - Edit
---

# /mhc-status — Light orientation, filesystem-only

Show the current MHC session state and orient the user in the project. **Pure filesystem read-only** — reads `.mhc-config.json` + `session_topology.yaml` + `adapt.md` frontmatter + `CLAUDE.md` prose. No MCP server calls, no network requests, no external dependencies.

This is the natural entry point for contextual session openings: when the user says *"cosa facciamo oggi"*, *"a che punto siamo"*, *"dove eravamo rimasti"*, *"stato del progetto"*, *"orientami"*, *"riassumi"*, *"come va"*, *"what is the state"*, *"where were we"*, *"orient me"*, *"recap"* — invoke this skill organically rather than improvising an answer from memory.

## What to display

### Block 1 — Current session

Read `.mhc-config.json` from the project root.

- **Session ID** and **start time** (from `current_session.id` or `current_session.sid` + `current_session.started_at`)
- **`continues_from`** (previous SID, if any)
- **`goal`** — mark `(unset)` if empty; this prompts the user to set one
- **`inputs`** and **`artifacts_produced`** for this session if known

If `.mhc-config.json` does not exist: surface this — *"No `.mhc-config.json` in this project. Run `/mhc-onboard` to set up MHC-C on this project."* Stop here.

### Block 2 — Infrastructure

- **Project name** (from `project_name`)
- **Project nature** (from `project_nature`, e.g. `design_workspace`, `governance_portfolio`, `research_workspace`)
- **Language** (from `language`)
- **`onboarding` block** — show `completed: true/false`, `case` (A/B/C/D/E), `mhc_version` if present. If the `onboarding` block is missing entirely, surface this explicitly: *"Onboarding not recorded — run `/mhc-onboard` to set up the project."*

### Block 3 — Recent sessions

From `session_history` (or `sessions` array) in `.mhc-config.json`: show the last 3-5 entries with their SID + start_time + ended_at (if closed) + brief goal/topic if present.

If the day has been heavy (many sessions), say so briefly: *"Heavy day — N prior sessions today."*

### Block 4 — Session topology

If `session_topology.yaml` exists at the project root: read it and show this session's entry — goal, inputs, artifacts_produced.

### Block 5 — Adaptation summary

If `adapt.md` exists at the project root: parse its YAML frontmatter and show:

- **Schema version** (e.g., `1`)
- **Primary output location**
- **Artifact types**: list those marked `present`
- **Skills enabled**: list those marked `enabled`

Note any custom artifact types defined in the body.

### Block 6 — Drift check (NEW — MHC-C exclusive)

After displaying Blocks 1-5, perform a **sync check** between `adapt.md` frontmatter and `CLAUDE.md` prose to detect drift in the **skills list**.

#### Logic

1. Parse the `skills:` block from `adapt.md` frontmatter — this is the **canonical** list of enabled/disabled/redirected skills (machine-readable, atomically editable). Build a sorted list of skills marked `enabled`.

2. Read `CLAUDE.md` from the project root. Locate the `## Available skills` section (or `## Skills disponibili` for Italian projects). Extract the skill names listed there (typically in inline code: `` `/mhc-trace`, `/mhc-note`, ... ``). Build a sorted list.

3. **Compare** the two lists:
   - If identical: no drift, **silent** (do not surface a warning).
   - If divergent: surface the drift to the user with the diff:

     > **Drift detected** between `adapt.md` (canonical) and `CLAUDE.md` (derived).
     >
     > - In `adapt.md` enabled but missing from `CLAUDE.md`: `<list>`
     > - In `CLAUDE.md` but not enabled in `adapt.md`: `<list>`
     >
     > Should I regenerate the `CLAUDE.md` skills section from `adapt.md`? (canonical → derived) [Y / n]

4. **On Approve (Y)**:
   - Read the full `CLAUDE.md`
   - Locate the `## Available skills` section (start) and the next `##` heading or `---` separator (end)
   - Build a new section content:
     ```markdown
     ## Available skills

     `/mhc-start`, `/mhc-status`, `/mhc-trace`, `/mhc-note`, `/mhc-pdl`, `/mhc-output`, `/mhc-modlog`, `/mhc-end`, `/mhc-onboard`

     ---
     ```
     (substituting the actual list of `enabled` skills from `adapt.md` frontmatter, prepending `/` to each skill name)
   - Use the `Edit` tool to replace ONLY that section — leave the rest of `CLAUDE.md` untouched
   - Confirm to the user: *"`CLAUDE.md` skills section regenerated from `adapt.md`. Lines changed: <N>. Other content preserved."*

5. **On Cancel (n)**:
   - Proceed but emit a **loud warning**: *"Drift NOT resolved. The skills declared in `CLAUDE.md` and `adapt.md` diverge. Claude's auto-loaded context (from `CLAUDE.md`) may reference skills not enabled here, or omit skills that ARE enabled. Run `/mhc-status` again later to revisit."*

**Why this matters:** the skills list lives in two places by design — `adapt.md` frontmatter is canonical (atomically editable YAML), `CLAUDE.md` prose is derived (human-readable + Claude auto-loaded). The two have decoupled lifecycles (onboarding is one-shot; `adapt.md` updates happen mid-project; `CLAUDE.md` is read at every Claude session start). `mhc-status` catches drift organically because it is triggered on contextual session opening — the most natural place for the user to want orientation.

### Block 7 — After displaying

If `goal` is `(unset)` for the current session, ask the user once if they want to set it (Rule 2 — Salomone). Otherwise, hand back to normal session work.

## Example output

```
MHC Session Status — example_project

Current session: SID-20260527-102449 (started 2026-05-27 10:24:49)
- Continues from: SID-20260526-172143
- Goal: Implementare MHC-C Phase 1
- Inputs / artifacts_produced: 3 captured

Project: example_project · nature: governance_portfolio · language: it
Onboarding: completed (case A, v0.1)

Recent sessions (last 3):
- SID-20260527-102449 (today, open)
- SID-20260526-172143 (yesterday, 4h28m, closed, exported)
- SID-20260526-011753 (yesterday, 2h12m, closed, exported)

session_topology.yaml: this session's goal is "MHC-C Phase 1 implementation"

adapt.md: schema v1 — primary_output_location: _org/
- Artifact types present: notes, traces, modlogs, decision_log, synthesis
- Skills enabled: mhc-start, mhc-status, mhc-trace, mhc-note, mhc-modlog, mhc-onboard

Drift check: no drift between adapt.md and CLAUDE.md. ✓

Ready to continue.
```

## Boundaries (what /mhc-status does NOT do)

This skill is **light orientation only**. It does NOT:

- Read `decision_log.md` or any decision log content (that's domain-specific governance data; if a project has it, it's not surfaced by this skill — MHC-C standalone treats it as opaque project content).
- Read `synthesis/` or any open-tension content (same rationale).
- Surface contradictions, audit chain status, or cross-session reconciliation against external sources (that is the value-prop of MHC-H — the harness that adds cross-session memory + audit chain + drift detection unlocked by configured authority).
- Call any MCP server, make any network request, or invoke any external tool. Pure local filesystem read.
- Modify any file **except** the `CLAUDE.md` skills section, and only on explicit user approval after surfacing the drift.

If you want the deeper governance picture (decision_log surfacing, open tensions, cross-session memory, audit chain verification), that requires MHC-H configured — see `MicheleLoi/MHC-H` (in development).

---

*MHC-C `/mhc-status` — light orientation, filesystem-only. Originally part of MHC-W (template `MHC-W/templates/skills/mhc-status/SKILL.md`), extracted + retrofitted to MHC-C 2026-05-27. Decalogo intact.*
