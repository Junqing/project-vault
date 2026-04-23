---
name: projectvault-init-repo
description: >
  Use when working in ProjectVault and the prompter uses /vault-init [slug] or asks to move an
  idea to a real repo. Reads the idea's design files, generates ProjectInitialize.md, and
  updates the idea's status to Building.
---

# ProjectVault: Init Repo Skill

Generate `ProjectInitialize.md` for a finished design and mark the idea ready to build.

## Step 0: Identify the Slug

Extract `[slug]` from the trigger command. If no slug is given, ask: "Which idea are you ready to build? (use the slug from `README.md`)"

---

## Step 1: Verify Readiness

Read the following files for the idea (fail loudly if any are missing):
- `ideas/[slug]/Overview.md` — confirm status is not already `Building`
- `ideas/[slug]/Design-Spec.md` — required; contains the source of truth
- `ideas/[slug]/Stack.md` — required; needed for stack setup commands
- `ideas/[slug]/Architecture.md` — read if present; used for infra diagram summary

If `Design-Spec.md` is missing, stop and say:
> "`Design-Spec.md` not found for `[slug]`. Run `/design [slug]` first to complete the design before initialising a repo."

---

## Step 2: Generate `ideas/[slug]/ProjectInitialize.md`

Write the file using this structure, populated entirely from the idea's existing design files:

```markdown
---
tags: [projectvault, [slug], init]
type: init
created: YYYY-MM-DD
---

# [Idea Name] — Repo Initialisation

Everything needed to bootstrap a real code repository from this design.

---

## 1. Repo Bootstrap

```bash
gh repo create [slug] --private --clone
cd [slug]
git commit --allow-empty -m "chore: initial commit"
```

---

## 2. Copy Design Docs

```bash
mkdir -p docs
cp -r /path/to/ProjectVault/ideas/[slug]/* docs/
```

Files copied:
| File | Purpose |
|------|---------|
| `Overview.md` | Elevator pitch, goals, non-goals |
| `Stack.md` | All tech decisions with comparison tables |
| `Architecture.md` | System diagrams, data model, API tables |
| `Phases.md` | Phased build plan |
| `Design-Spec.md` | Source of truth — read this first |
| `Research/` | Interactive HTML artifacts from brainstorming |

---

## 3. First Steps After Copy

Open `docs/Design-Spec.md`. Start with Phase 1 from `docs/Phases.md`.

Suggested first prompt in the new repo:
> "I'm starting implementation of [Idea Name]. The design is in `docs/Design-Spec.md` and the phased plan is in `docs/Phases.md`. Start with Phase 1."

---

## 4. Stack Setup

[Derived from Stack.md — list exact commands to set up the chosen stack]

Example (fill from Stack.md):
```bash
# Backend
python -m venv .venv && source .venv/bin/activate
pip install fastapi uvicorn[standard] sqlalchemy psycopg2-binary alembic

# Frontend
npx create-next-app@latest frontend --typescript --tailwind
```

---

## 5. Environment Variables

| Variable | Purpose |
|----------|---------|
[Derived from Design-Spec.md Infrastructure section — list all env vars]

---

## 6. Related

- [[Overview]] — elevator pitch and goals
- [[Stack]] — all tech decisions
- [[Architecture]] — system design and data model
- [[Design-Spec]] — consolidated implementation spec
```

**Important:** Populate every section from the actual design files. No placeholders — if a section cannot be derived from the existing docs, note what is missing and why.

---

## Step 3: Update Status

**In `ideas/[slug]/Overview.md`:** Change `status: Exploring` (or current status) → `status: Building`

**In `README.md`:** Update the idea's row — change the Status cell to `Building`.

---

## Step 4: Confirm

> "`ProjectInitialize.md` created for `[slug]`. Status updated to Building.
>
> Next: copy `ideas/[slug]/` → `docs/` in your new repo and follow the instructions in `ProjectInitialize.md`."
