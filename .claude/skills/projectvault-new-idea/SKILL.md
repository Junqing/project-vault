---
name: projectvault-new-idea
description: >
  Use when working in ProjectVault and a new project idea is being described or explored.
  Handles the full brainstorm-to-spec workflow using vault conventions, Stack-Preferences,
  and Prompter-Profile. Do NOT invoke superpowers:brainstorming in this context.
---

# ProjectVault: New Idea Design Skill

You are running the full single-idea brainstorm-to-spec workflow for the ProjectVault. This skill replaces `superpowers:brainstorming` entirely. Do not invoke that skill.

## Step 0: Load Context

Before asking any questions, read:
1. `_shared/Stack-Preferences.md`
2. `_shared/Prompter-Profile.md`

If `Stack-Preferences.md` is missing, run the stack setup flow (see below) before continuing.
If `Prompter-Profile.md` is missing, run the guest setup flow (see below) before continuing.
If both are missing, run stack setup first, then guest setup.

---

## Step 1: Richness Check

Assess how much the prompter has already provided. Look for these 4 signals in their message:
- **Slug**: is there a clear short name for the idea?
- **Audience**: who is it for?
- **Interaction model**: how does the user interact with it?
- **Constraints**: any tech, scope, or platform constraints stated?

| Signals present | Q&A approach |
|-----------------|-------------|
| All 4 | Skip questions. Confirm inferred scope in 1 message, get approval, proceed. |
| 3 of 4 | Max 2 clarifying questions. |
| 2 or fewer | Ask up to `max_clarifying_questions` (from profile, default: 4), one at a time. |

Never exceed the profile's `max_clarifying_questions` cap.

---

## Step 2: Ask Clarifying Questions

Ask one question at a time. Prefer multiple-choice options when possible. Stop as soon as you have enough to propose approaches.

**Before each question, check:**
- Is this topic in the profile's `skip_questions` list? If yes, skip it.
- Is `assumed_stack: true`? If yes, skip all stack/infra questions.

**Adapt framing based on `technical_depth`:**

| technical_depth | Infra questions | Stack questions |
|-----------------|----------------|-----------------|
| senior / mid | Ask directly: "ECS or Lambda?" | Ask directly: "FastAPI or Django?" |
| junior | Ask as tradeoffs: "Always-on vs pay-per-request?" | Ask as outcomes: "One language or best-fit per layer?" |
| non-technical | Suppress infra questions entirely | Ask as outcomes: "Do users need accounts?" |

---

## Step 3: Propose 2–3 Approaches

Present architectural options. Always lead with Approach A pre-populated from `Stack-Preferences.md` defaults. Include trade-offs for each. Make a clear recommendation with reasoning.

Wait for the prompter to choose before proceeding.

---

## Step 4: Present Design Sections

Walk through the design one section at a time:
1. Architecture overview
2. Data model
3. Frontend components + routes
4. API endpoints + deployment

After each section, ask: "Does this look right?" Wait for approval before moving on.

**Visual companion:** If `prefer_visuals: true` in profile (and not already offered this session), offer the visual companion once before starting design sections:
> "Some of what we're working on might be easier to explain in a browser — mockups, diagrams, architecture views. Want to try it? (Requires opening a local URL)"

Wait for response. Only offer once.

---

## Step 5: Create Files (in this exact order)

### 1. `ideas/[slug]/Overview.md`
```markdown
---
tags: [projectvault, [slug]]
type: index
status: Exploring
created: YYYY-MM-DD
---

# [Idea Name]

## Elevator Pitch
[One paragraph]

## Goals
- [goal 1]
- [goal 2]

## Non-Goals
- [non-goal 1]

## Status
Exploring
```

### 2. `ideas/[slug]/Stack.md`
Use this structure for each significant decision:
```markdown
## Decision: [e.g., Database]

**Chosen:** [option]

| Option | Pros | Cons |
|--------|------|------|
| ...    | ...  | ...  |

**Why:** [reasoning]
```
Include a Standing Decisions table at the end for choices that match Stack-Preferences defaults (no table needed, just list them with a note).

### 3. `ideas/[slug]/Architecture.md`
- Mermaid `flowchart LR` for system overview (if more than 1 component)
- Mermaid `erDiagram` for data model
- Mermaid `flowchart TD` for component tree
- JSON code block for any JSONB or complex schema shapes
- API tables: Method | Path | Auth | Notes
- Frontend routes table: Route | Type (SSR/CSR) | Component | Notes

### 4. `ideas/[slug]/Phases.md`
```markdown
## Phase 1: [Name]
**Goal:** [one sentence]
- [ ] [feature/component]

## Phase 2: [Name]
...
```

### 5. `ideas/[slug]/Design-Spec.md`
Nine required sections:
1. Product Overview
2. Users & Roles
3. Core Model
4. Data Model
5. API
6. Frontend
7. Infrastructure
8. Phased Build
9. Out of Scope

### 6. `ideas/[slug]/Research/*.html`
Save all interactive artifacts from brainstorming as standalone static HTML files. Each file must be fully self-contained (no server required), respond to dark/light mode via CSS `prefers-color-scheme`, and retain click interactivity.

Name files semantically: `layout-types.html`, `design-architecture.html`, etc.

### 7. Update `README.md`
Add a row to the Ideas table:
```
| [[ideas/[slug]/Overview\|[slug]]] | Exploring | [stack summary] | [one-line description] |
```

---

## Step 6: End State

After all files are created:
> "Design complete. All files saved in `ideas/[slug]/`. When you're ready to build, say `/vault-init [slug]`."

Do NOT invoke `writing-plans`. This vault is design-only. Implementation handoff happens via `ProjectInitialize.md`.

---

## Stack Setup Flow

Triggered when `_shared/Stack-Preferences.md` is missing. Ask these questions one at a time, then write the file before continuing.

1. "What's your primary backend language?" (e.g., Python, Go, TypeScript, Java)
2. "What's your preferred web framework for that language?" (e.g., FastAPI, Django, Express, Spring)
3. "What's your default database?" (e.g., PostgreSQL, MySQL, SQLite, DynamoDB)
4. "Where do you typically deploy?" (e.g., AWS, GCP, Azure, Fly.io, self-hosted)
5. "Any preferred frontend approach?" (e.g., Next.js, plain HTML/HTMX, React SPA, no preference)
6. "Any preferred IaC tool?" (e.g., Terraform, CDK, Pulumi, none)

Write the results to `_shared/Stack-Preferences.md`:

```markdown
---
tags: [projectvault, stack]
type: reference
created: YYYY-MM-DD
---

# Stack Preferences

Default technology choices for this ProjectVault. Override per-idea in `Stack.md`.

| Domain | Choice | Notes |
|--------|--------|-------|
| Backend language | [answer] | |
| Web framework | [answer] | |
| Database | [answer] | |
| Cloud / deployment | [answer] | |
| Frontend | [answer] | |
| IaC | [answer] | |
```

---

## First-Time Setup Flow

Triggered when `Prompter-Profile.md` is missing. Ask these 4 questions (one at a time), then write the profile before doing anything else.

1. "What's your name?"
2. "Are you technical? (yes / somewhat / no)"
   - yes → `technical_depth: senior`
   - somewhat → `technical_depth: mid`
   - no → `technical_depth: non-technical`
3. "Do you have a preferred tech stack, or would you like to be asked about it for each idea?"
   - have preferences → ask them to briefly describe their stack, set `assumed_stack: true` and capture choices in `stack_overrides`
   - ask each time → `assumed_stack: false`
4. "When you mention an idea, do you want a full detailed design right away, or just a quick stub to come back to later?"
   - full → `rapid_mode_default: false`
   - stub → `rapid_mode_default: true`

Write the profile to `_shared/Prompter-Profile.md` using the schema below, then resume the current request.

```yaml
---
name: [name]
role: owner
technical_depth: [from Q2]
assumed_stack: [from Q3]
skip_questions: []
rapid_mode_default: [from Q4]
prefer_visuals: true
max_clarifying_questions: 4
stack_overrides: {}
---

[Name]'s profile for ProjectVault brainstorming sessions.
```
