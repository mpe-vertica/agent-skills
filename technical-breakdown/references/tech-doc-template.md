# Technical description document template

Title: `Technical description – <feature name>`. Attach to the requirement
issue. Headings are shown in English; follow the requirement issue's language.

```markdown
Supplements the acceptance criteria in [ABC-xxxx](link) with the technical
decisions that need approval. Scope and UI are in the spec and not repeated here.

## 1. Starting point
* <2–4 bullets: the architectural facts the decisions rest on, e.g. which
  service serves the page, generated vs hand-written client, what exists in the
  CMS.>

## 2. Decisions
1. **<Decision as a short statement.>** <Why, in one or two sentences. Who
   confirmed it if not the team (PO/UX).>
2. ...
<Only decisions that weren't obvious from the spec. 4–8 is typical.>

## 3. Risks
| # | Risk | Handling |
| -- | -- | -- |
| R1 | <What could go wrong> | <Who/what handles it, or "Accepted"> |

## 4. Tasks
Parent: [ABC-xxxx](link) <title>

| Issue | Task | Blocked by |
| -- | -- | -- |
| [ABC-xxxx](link) | <title> | – |
```

Self-check before saving:
- Would the architect learn anything they couldn't get from the spec? If a
  bullet just restates the spec, cut it.
- Is there a file path, field list or code snippet? Move it to an issue.
- Is every decision's *why* stated, and is it clear who decided?
- Does the sub-issue table match Linear (titles, blockers)?
