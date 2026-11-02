# agent-skills

[Agent Skills](https://agentskills.io) for coding agents. Each skill is a folder
with a `SKILL.md` (YAML frontmatter + instructions) and optional `references/`.
The same folder works unchanged in Claude Code, OpenCode and other agents that
support the Agent Skills format.

| Skill | What it does |
| -- | -- |
| [`technical-breakdown`](technical-breakdown/SKILL.md) | Turns a requirement/spec issue in Linear into backend/frontend sub-issues under a new parent, a branching strategy (what ships to `main` alone, whether an integration branch is needed), blocker relations, and a short technical description doc for architect sign-off. Has a dry-run mode that writes everything to a local `breakdown.md` instead. |

## Install

Clone the repo and copy a skill folder (keep the folder name, it must match
`name` in `SKILL.md`) to where your agent looks for skills:

```sh
git clone https://github.com/mpe-vertica/agent-skills.git
cp -r agent-skills/technical-breakdown <skills-dir>/
```

| Agent | Personal (all projects) | Per project |
| -- | -- | -- |
| Claude Code | `~/.claude/skills/` | `.claude/skills/` |
| OpenCode | `~/.config/opencode/skills/` | `.opencode/skills/` |
| Other Agent Skills agents | `~/.agents/skills/` | `.agents/skills/` |

OpenCode also reads `~/.claude/skills/` and `.claude/skills/` (and the
`.agents/` paths), so a single copy in a `.claude/` location covers both Claude
Code and OpenCode.

## technical-breakdown

### What

Turns a requirement/spec issue in Linear into a technical breakdown:

- a new parent issue grouping `Backend`/`Frontend`-labelled sub-issues with
  blockers, in the given project/milestone
- a **branching strategy**: what is additive/inert and can go straight to `main`
  (BE fields, CMS migrations, shared UI components, opt-in props), and whether
  anything must go live together. Only then is an **integration branch**
  proposed, using the parent issue's branch
- a short **technical description** Linear document on the requirement issue,
  for the lead architect to sign off on

### How it works

1. **Investigate:** data source (generated vs hand-written types, backend),
   reusable components, CMS content model, Figma, page variants (e.g. B2C/B2B),
   CI unused-code checks (e.g. knip entries).
2. **Decide and plan:** non-obvious decisions are recorded with who decided
   them (team/PO/UX); the plan is approved before anything is written to Linear.
3. **Write to Linear** in dependency order, patch cross-links and verify.
4. **Technical description:** starting point, decisions with their *why*, risks
   and the sub-issue table, scoped to what the spec leaves open.
5. **Keep in sync:** when a decision changes later, every issue, the document
   and the plan are updated together.

Templates for the issue body and the document are in
[`technical-breakdown/references/`](technical-breakdown/references/).

### Usage

The skill triggers on requests like these:

```
break ABC-1234 down into backend and frontend tasks under milestone Development in project X
write a technical description for ABC-1234
```

#### Dry run

Ask for a dry run (e.g. "dry run ABC-1234") to get the full plan, the issues and
the document in a local `breakdown.md`, with Linear used read-only. This is
useful for trying the skill out or reviewing a breakdown before it goes live.

### Prerequisites

- **Linear MCP:** required to write the breakdown to Linear (read-only in a dry run).
- **Read access to backend repos** (e.g. `gh`) if the backend lives elsewhere.
- **Figma MCP (optional):** for designs. The desktop MCP only reads the file in
  the active tab.

The investigation checklist assumes a typical web stack (API + frontend, optional
CMS, CI unused-code checks like knip). Skip the steps that don't apply.

## License

MIT
