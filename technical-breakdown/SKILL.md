---
name: technical-breakdown
description: >-
  Break a requirement/spec issue in Linear into a technical breakdown:
  backend and frontend sub-issues under a new parent, a branching strategy that
  decides what can be released to main on its own and whether anything must ship
  together via an integration branch, blocker relations, and a short technical
  description document for the lead architect to sign off on. Use this whenever
  the user asks to break down, split up, plan or slice a Linear issue (e.g.
  "break ABC-1234 down into BE and FE tasks", "create the dev tasks for project
  X", "what tasks do we need for this spec", "write a technical description for
  ABC-1234"), even if they don't mention branches or documents explicitly.
license: MIT
compatibility: Requires a Linear MCP server. Figma MCP and gh are optional.
---

# Technical breakdown of a requirement issue

The goal is to turn a functional spec (usually a Linear issue written by PO/UX)
into a set of Linear issues a developer can pick up without re-doing the
analysis, plus a short document that lets the lead architect approve the
non-obvious decisions. The breakdown is only as good as the investigation behind
it, so most of the effort goes into understanding the codebase and resolving
open questions *before* anything is written to Linear.

Linear writes are outward-facing and noisy to undo (notifications, relations,
IDs). So: investigate → agree on a plan with the user → write to Linear →
verify. Never create issues before the user has approved the plan.

## Dry-run mode

Run in dry-run mode when the user asks for it ("dry run", "just the plan",
"don't touch Linear", "what would you create"), or when the skill is being
tested or evaluated against an existing issue. In dry-run mode:

- Linear and other external systems are **read-only**: no issue, document,
  comment, relation, label or status writes. Reading the requirement issue,
  designs, the codebase and backend repos is fine and expected — the
  investigation is the point.
- Do Phases 1–2 as normal, then instead of Phases 3–4 write the result to a
  local `breakdown.md` (the path the user gives, otherwise a scratch/temp
  directory — not inside the repo, so it doesn't end up in a commit)
  containing, in this order:
  1. The plan: findings, decisions (with who decided / still to confirm),
     branching strategy.
  2. The issues exactly as they would be created: title, label, target branch,
     blockers, and the full body. Use placeholders `NEW-1`, `NEW-2`, … for
     issues that don't exist yet, with the parent as `NEW-0`.
  3. The technical description document exactly as it would be saved.
- Questions you would normally ask the user: still ask them interactively. If
  running non-interactively, pick the most defensible default, mark it
  `ASSUMED:` in the plan, and carry on.
- End by telling the user where the file is and that nothing was written to
  Linear. Going live later means running Phases 3–4 from that file, replacing
  placeholders with real identifiers.

## Inputs to establish

Get these from the user's message or ask once, together (with the agent's
question tool if it has one, e.g. AskUserQuestion), for anything missing:

- **Requirement issue** (e.g. `ABC-1234`). Read description, comments,
  attachments, linked designs, project and relations.
- **Placement:** team, project, milestone, initial status (default `Backlog`).
- **Reference breakdown** the user liked, if any (a Linear project or issue) —
  use it to calibrate granularity and tone, not as a template to copy.
- **Release constraints:** is a feature flag wanted? Must everything visible go
  live at once? Does CI run unused-code checks (e.g. knip) that would fail on
  code that isn't wired up yet?
- **Labels:** which labels distinguish backend from frontend work in this team
  (default `Backend` / `Frontend`). Check they exist.

## Conventions

- **Labels, not prefixes.** Every sub-issue gets exactly one of the backend /
  frontend labels. Titles carry no `BE:`/`FE:` prefix — the label already says
  it, and the prefix just eats title space.
- **Titles** say what is delivered, in the language of the requirement issue,
  e.g. "Certificate modal + analytics tracking".
- **Parent issue** is new, not the requirement issue. It is a sibling of the
  requirement (related-to link) and groups all sub-issues. The requirement issue
  stays the spec. If the branching strategy calls for an integration branch,
  the parent's `gitBranchName` is that branch.
- **Branch names** come from each issue's `gitBranchName`, never invented.
- Issue bodies and the document follow the requirement issue's language.

## Phase 1 — Investigate

Work out, with evidence (file paths), the answer to each of these:

1. **Where does the data come from?** Which API/endpoint feeds the screen.
   Is the frontend client generated (OpenAPI/GraphQL codegen) or hand-written
   types? If hand-written, the contract must be spelled out identically in the
   BE and FE issues. If the backend lives in another repo, read it read-only
   (e.g. via `gh api` or a local clone) rather than guessing its behaviour.
2. **What already exists** that the feature can reuse: shared UI components,
   modals, accordions, rich text renderers, tracking helpers,
   experimentation/flags, structured-data builders, CMS models. Prefer
   extending over new components; when an existing component is shared, check
   its current consumers so the change can't alter them (make new behaviour
   opt-in).
3. **CMS/content model** (if there is a CMS): existing content types, how
   migrations are numbered and run, which environments/spaces CI migrates,
   locales. Content prerequisites (entries that must exist before editors can
   work) are part of the breakdown.
4. **Design:** read the design frames (e.g. via the Figma MCP). The desktop
   Figma MCP only reads the file in the active tab — if a node isn't found, ask
   the user to open that file or send the exact link rather than guessing. Map
   each design element to an existing component, or call out a genuinely new
   one. Flag designs that use deprecated components.
5. **Variants of the page** that need the feature: e.g. B2C vs B2B templates,
   bundles, white-label or partner sites, locales/markets. Decide explicitly
   which are in scope.
6. **CI constraints:** know which code is allowed to land unused. With knip or
   similar, read its config to see which files/exports are entry points —
   exports of a shared package listed as entries can land on main unused; app
   code usually cannot.

If a tool fails (expired MCP token, missing access), record it as an open risk
with a concrete verification step in the relevant issue instead of stalling.

## Phase 2 — Decide and plan

Collect every decision that is **not obvious from the spec** — those are what
the architect needs to see and what PO/UX may need to confirm. Typical ones:
which side enriches data (BE vs FE), data model direction (e.g. child references
parent so it's queryable), scope of a flag, required vs optional CMS fields,
which variants are covered. Ask the user about genuine forks; record who decided
(user, PO, UX) since the document will cite it.

### Branching strategy

Classify each task:

- **Standalone → `main`:** additive, inert until used, and passes CI on its own.
  Typical: additive API response fields, CMS migrations, a new component in a
  shared package whose exports are entry points, opt-in props on shared
  components.
- **Integration branch:** anything that changes what users see, or app code that
  would be flagged as unused until the rest lands, *and* has to go live together
  with other tasks. PRs target the parent's branch.

The integration branch is an outcome of this classification, not a given. If
every task can ship to `main` on its own (small features, or visible changes
that are fine to release one at a time, or gated by a flag the user chose to
use for that), the strategy is simply "all PRs to `main`" and there is no
integration branch. Propose the outcome and the reasoning; the user decides.

When there is an integration branch, state in the parent issue:
- How the integration branch is kept up to date with `main` (follow the team's
  convention; merging avoids rewriting history under open PRs).
- Release = merging the integration branch to `main`, gated on what must be in
  prod first (backend deployed, content created in all environments/locales).
- A feature flag does not automatically remove the need for the integration
  branch — ask. Flagged code can sit on main, but partially built UI behind a
  flag is still a choice the user should make.

### Granularity

One issue per independently reviewable unit with a clear owner. Split by
dependency boundary (data contract, content model, component, page wiring,
tracking/flag), not by file. Avoid issues so small they only make sense in the
same PR. A typical feature lands at 1–2 BE and 4–8 FE issues.

### Plan file

Write the plan (findings, decisions, branch strategy, issue table with target
branch and blockers, per-issue summary, open items) and get the user's approval
— via plan mode if the agent has one. Keep this file current for the rest of the
session; it is the source of truth when decisions change later.

## Phase 3 — Write to Linear

1. Create the parent issue (no BE/FE label) in project + milestone, related to
   the requirement issue. Body: purpose, the branching strategy (integration
   branch name and release checklist, if there is one), list of sub-issues
   (fill in after step 2).
2. Create sub-issues in dependency order (so blockers exist before they're
   referenced) with `parentId`, project, milestone, status, label and
   `blockedBy`. Use the body structure in `references/issue-template.md`.
3. Cross-references to issues created later can only be filled in afterwards —
   patch them in and double-check no issue links to itself.
4. Verify: list the parent's children and check count, labels, milestone,
   status and blockers.

## Phase 4 — Technical description document

A Linear document attached to the **requirement issue**, for the lead architect
to sign off. See `references/tech-doc-template.md` for structure. Scope it
tightly:

- **Audience:** an architect who knows the platform but not this feature's
  analysis. They are approving decisions, not implementing.
- **Content:** only what the spec doesn't already say — the starting point
  (relevant facts about current architecture), the non-obvious decisions with
  the one-line *why* and who decided, risks with handling, and a table of all
  sub-issues with blockers.
- **Leave out:** restated acceptance criteria, UI descriptions, file paths,
  code, field-by-field models (those live in the issues), and the branch
  mechanics beyond one line.
- **Length:** fits on one screen-and-a-bit. If a section grows, it's probably
  implementation detail that belongs in an issue.

## Phase 5 — Keep it in sync

Decisions keep changing after the breakdown (PO/UX answers, review). When one
changes, update everywhere it appears in the same turn: every affected issue,
the document (only if it's at decision level), and the plan file. Search the
issues for the old wording rather than relying on memory, and report what was
changed where.
