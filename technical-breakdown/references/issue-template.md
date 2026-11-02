# Sub-issue body template

Write in the requirement issue's language. Headings are shown in English; keep
the structure and translate them if the requirement is in another language.

```markdown
**Repo:** `<repo>` (omit when it's the main repo for this team)
**Branch:** PR directly to `main` — <why it can ship alone>.
   | PR to the integration branch `<parent gitBranchName>` ([ABC-xxxx](link)), not `main`.
   (the latter only when the branching strategy has an integration branch)

## Goal
<What is delivered and why, 2–4 lines. Link the issues it builds on.>

## Tasks
* <Concrete steps with pointers: existing files/components/patterns to follow,
  with paths. Say what to reuse and what must not change.>
* <For a data contract shared between hand-written BE and FE types: the exact
  shape, identical in both issues.>

## Pointers
* <Reference implementations, migration numbers, config, gotchas found during
  investigation.>

## Acceptance criteria
* <Verifiable outcomes, incl. tests and what must remain unchanged.>

## Design
* <Design links (desktop/mobile) and design-system component names, if relevant.>
```

Guidance:
- An issue should be pickable by someone who wasn't in the analysis. Put
  findings where they're needed (e.g. "check the attribute type in the admin
  first") instead of only in the plan.
- State dependencies in prose *and* as Linear `blockedBy` relations.
- Unverified assumptions become an explicit first step, not a silent premise.
