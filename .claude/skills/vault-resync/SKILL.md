---
name: vault-resync
description: >
  Use when /vault-resync is invoked in ProjectVault, or when the user asks to
  resync, repair, or update the vault harness after workflow changes. Audits
  CLAUDE.md, skill files, README.md, and idea folder structures for drift, then
  proposes and applies fixes.
---

# ProjectVault: Vault Resync Skill

You are auditing and repairing the ProjectVault harness — CLAUDE.md, skills, README.md, and idea folders — for drift caused by incremental workflow changes.

---

## What "drift" means

Drift is any mismatch between:
- What CLAUDE.md says should exist (triggers, skill names, folder rules, status keys) vs. what actually exists on disk
- What README.md says about an idea vs. what that idea's `Overview.md` frontmatter says
- What a skill's frontmatter claims vs. what it does
- What CLAUDE.md's trigger table references vs. which skill files actually exist

---

## Audit Steps

Run all four audits, then present a single consolidated report before touching anything.

### Audit 1: Skill inventory

1. List all directories under `.claude/skills/`
2. For each, check that `SKILL.md` exists and has valid frontmatter (`name`, `description`)
3. Cross-reference against every skill name mentioned in CLAUDE.md (trigger table, Skill Overrides section, slash command table)
4. Flag:
   - Skills referenced in CLAUDE.md that have no matching directory
   - Skill directories that are not referenced anywhere in CLAUDE.md
   - Skill `name` fields that don't match their directory name

### Audit 2: README.md vs Overview.md status sync

1. Read `README.md` — parse every row in the Ideas table (slug, status, stack, description)
2. For each slug, read `ideas/[slug]/Overview.md` frontmatter
3. Flag any mismatch between the `status` field in README.md and the `status:` frontmatter in Overview.md
4. Flag any slug in README.md that has no matching folder under `ideas/`
5. Flag any folder under `ideas/` that has no row in README.md

### Audit 3: Idea folder completeness

For each idea folder under `ideas/`:
1. Check which files are present: Overview.md, Stack.md, Architecture.md, Phases.md, Design-Spec.md
2. Compare against the idea's status — use this matrix to determine what's expected:

| Status | Required | Expected (warn if missing) |
|--------|----------|----------------------------|
| Seed | Overview.md | — |
| Exploring | Overview.md | Stack.md |
| Designing | Overview.md, Stack.md | Architecture.md, Phases.md |
| Building | Overview.md, Stack.md, Architecture.md, Phases.md, Design-Spec.md, ProjectInitialize.md | — |
| Parked / Shipped | Overview.md | — |

Flag missing **required** files as errors. Flag missing **expected** files as warnings.

### Audit 4: CLAUDE.md self-consistency

1. Read CLAUDE.md — check the "Trigger Phrases" table and "Skill Overrides" section
2. For each skill name referenced, confirm the skill directory exists
3. Check that the Status Key in README.md matches the status values used in CLAUDE.md and idea Overview.md files
4. Flag any trigger phrases pointing to nonexistent skills

---

## Report Format

Present findings as a structured diff before doing anything. Group by severity:

```
## Vault Resync Report

### Errors (must fix)
- [SKILL-MISSING] CLAUDE.md references skill `foo-bar` but .claude/skills/foo-bar/ does not exist
- [STATUS-MISMATCH] README.md says hobby-learning-webapp=Exploring, Overview.md says Designing
- [FILE-MISSING] ideas/my-idea/ has status Building but is missing Design-Spec.md

### Warnings (worth reviewing)
- [SKILL-ORPHAN] .claude/skills/old-skill/ exists but is not referenced in CLAUDE.md
- [FILE-MISSING] ideas/my-idea/ has status Exploring but Stack.md is missing

### OK
- All trigger phrases resolve to existing skill files
- README.md row count matches ideas/ folder count
```

After presenting the report, ask:
> "Which of these do you want me to fix? (all / errors-only / specific items / none)"

Wait for confirmation before making any changes.

---

## Fix Behaviors

Only apply fixes that were explicitly approved.

| Issue type | Fix action |
|------------|------------|
| STATUS-MISMATCH | Ask which is correct (README or Overview), then update the other to match |
| SKILL-MISSING (referenced but absent) | Ask what the skill should do — do not create it silently |
| SKILL-ORPHAN (exists but unreferenced) | Ask whether to add a trigger to CLAUDE.md or delete the skill |
| FILE-MISSING (required) | Warn that the idea folder is in an inconsistent state — do not auto-create files |
| FILE-MISSING (expected/warning) | Note it but do not auto-create |
| README-MISSING (idea folder has no row) | Offer to add a row from the Overview.md content |
| README-ROW-DANGLING (row with no folder) | Offer to remove the row |

**Never silently create, delete, or overwrite files.** Always describe what you're about to do and wait for confirmation per fix.

---

## Applying Workflow Changes

If the user is running vault-resync because they have a new convention or workflow change (not just drift), do this first:

1. Ask: "What changed? (new skill, new folder rule, new status, trigger phrase update, other)"
2. Based on the answer, identify which files need updating:
   - New skill → create `.claude/skills/[name]/SKILL.md`, add trigger to CLAUDE.md
   - New folder rule → update CLAUDE.md's "Per-Idea Folder Rules" section
   - New status → update CLAUDE.md's status key AND README.md's Status Key section
   - Trigger phrase change → update CLAUDE.md's Trigger Phrases table
   - Skill rename → update skill directory name, skill `name` frontmatter, and all CLAUDE.md references
3. Then run the full audit to catch any secondary drift.

---

## End State

After all approved fixes are applied:
> "Resync complete. [N errors fixed, M warnings noted]. Run /vault-status to confirm the current state."
