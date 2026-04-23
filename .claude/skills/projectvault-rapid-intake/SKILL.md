---
name: projectvault-rapid-intake
description: >
  Use when working in ProjectVault and the prompter lists multiple project ideas at once,
  or uses /intake. Creates Overview.md stubs (status: Seed) for each idea. Never creates
  Stack.md, Architecture.md, Phases.md, or Design-Spec.md — only Overview stubs.
---

# ProjectVault: Rapid Intake Skill

You are running batch idea capture mode. The goal is to quickly record multiple ideas as stubs so the prompter can come back and design each one in depth later.

## HARD GATE

**Never create any of the following in this skill:**
- `Stack.md`
- `Architecture.md`
- `Phases.md`
- `Design-Spec.md`
- `Research/` files

Only `Overview.md` stubs are created. Any design work happens in a separate session using `/design [slug]`.

---

## Step 0: Load Context

Read `_shared/Stack-Preferences.md` and `_shared/Prompter-Profile.md` if they exist.
- If `Stack-Preferences.md` is missing: run the stack setup flow from `projectvault-new-idea` first.
- If `Prompter-Profile.md` is missing: run the guest setup flow from `projectvault-new-idea` first.
- If both are missing: stack setup first, then guest setup.

---

## Step 1: Identify Ideas

Extract the list of ideas from the prompter's message. If any idea is vague (no name, no description), ask a single follow-up to clarify that idea only. Move through the list quickly.

---

## Step 2: Per-Idea Questions (max 3 per idea)

For each idea, ask at most 3 questions if the information isn't already in the message:
1. **Slug**: what's a short kebab-case name? (infer it yourself if obvious, confirm only if ambiguous)
2. **Pitch**: one sentence — what problem does it solve?
3. **Audience**: who is it for?

If the prompter's original message already answers these, skip the questions for that idea.

---

## Step 3: Create Overview.md Stubs

For each idea, create `ideas/[slug]/Overview.md`:

```markdown
---
tags: [projectvault, [slug]]
type: index
status: Seed
created: YYYY-MM-DD
---

# [Idea Name]

## Elevator Pitch
[One sentence from intake]

## Audience
[Who it's for]

## Status
Seed — stub only. Say `/design [slug]` to start full design.
```

---

## Step 4: Update README.md

Add a row for each new idea to the Ideas table in `README.md`:

```
| [[ideas/[slug]/Overview\|[slug]]] | Seed | TBD | [one-line pitch] |
```

---

## Step 5: Summary

End with a summary table:

```
| Slug | Pitch | Status |
|------|-------|--------|
| [slug-1] | [pitch] | Seed |
| [slug-2] | [pitch] | Seed |
...
```

Then:
> "All ideas captured. Say `/design [slug]` to start the full design flow for any of them."
