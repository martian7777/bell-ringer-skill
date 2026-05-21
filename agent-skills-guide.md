# Agent Skills: Creating, Installing, and Sharing Reusable Agent Context

> Stop re-explaining your project's tribal knowledge every time you open a new chat. Skills are the "npm moment" for AI context: install once, let the agent load on demand.

This guide covers what skills are, how to author one, how to install and manage them with the `npx skills` CLI, and how publishing works in practice. It targets engineers who already use an agentic coding tool (Claude Code, Codex, Cursor, OpenCode, etc.) and want a reusable, versionable library of context instead of pasted prompts.

---

## Why skills exist

Models know general programming and frameworks. They don't know:

- Your folder structure
- Your design system
- Your deployment rules
- What "good" means in your org

You have two ways to give them that knowledge:

| Use AGENTS.md (passive) | Use a skill (active) |
|---|---|
| Applies to nearly every task | Specialized workflow |
| Must never be ignored | Used occasionally |
| Project-wide rules and guardrails | Reusable across repos and teams |
| Examples: "always use strict TS", security rules | Examples: "deploy to Vercel", "write a PRD" |

Pick passive for things that apply all the time. Pick a skill for workflows the agent should load only when relevant.

---

## Architecture

**Skill vs. skill package.** A *skill* is a folder containing a `SKILL.md` (plus optional supporting files). A *skill package* is a repo (or directory) containing one or more skills.

**Progressive disclosure.** Skills are designed so agents don't load everything up front:

1. **Metadata** (name + description) is read early so the agent can decide if the skill is relevant.
2. **Full instructions** (the body of `SKILL.md`) load only when the agent activates the skill.
3. **Resources** (scripts, references, assets) load only when the body tells the agent to use them.

This keeps the context window clean and lets you ship deep, detailed skills without bloating every conversation.

---

## Anatomy of a skill

A skill is a directory containing at minimum a `SKILL.md`:

```
my-skill/
└── SKILL.md
```

`SKILL.md` must start with YAML frontmatter. Required fields:

- `name` — unique identifier, lowercase with hyphens allowed
- `description` — what the skill does **and when to use it**

Optional fields (support varies; check your agent's docs):

- `metadata` — arbitrary key/value data; a good home for `version` (semver) and `internal: true` to hide from discovery
- `allowed-tools` — restricts which tools the skill can use (Claude Code, OpenCode, Codex, Cursor support it; **Kiro CLI and Zencoder do not**)
- Agent-specific extensions like `context: fork` (Claude Code only) and `hooks`

Optional supporting directories — use these to keep `SKILL.md` short and load detail on demand:

- `scripts/` — executable helpers
- `references/` — longer docs, checklists, templates
- `assets/` — static templates, sample configs, diagrams

### The description is the trigger

Agents decide whether to activate a skill primarily from its `description`. Treat it like a routing rule, not a title.

> **Bad:** "Helps with PDFs."
>
> **Good:** "Extract text and tables from PDFs, fill forms, merge documents. Use when the user mentions PDFs, forms, scanning, or document extraction."

More examples of the same pattern:

> "Deploy a Next.js app to Vercel, including preview deploys and env var sync. Use when the user mentions Vercel, deploy, preview URL, or `vercel.json`."

> "Review a React PR for performance regressions: re-renders, memoization, bundle size. Use when the user asks for code review on React components or mentions performance."

If you only polish one thing in a skill, polish the description.

---

## A complete example

Here is a small but real skill — a PR description writer — end to end.

**Directory:**

```
pr-description-writer/
├── SKILL.md
├── scripts/
│   └── collect-diff.sh
└── references/
    └── template.md
```

**`SKILL.md`:**

```markdown
---
name: pr-description-writer
description: Draft a PR description from the current branch's diff and commits.
  Use when the user asks to write, draft, or improve a PR description, or
  mentions "PR body", "pull request summary", or "describe this branch".
metadata:
  version: 0.2.0
---

# PR Description Writer

## When to use
- User has a branch ready to push or a PR already open and wants a description.
- Skip if the user wants commit messages (different workflow).

## Inputs
- Current branch name and base branch (default: `main`).
- Optional: linked issue number.

## Procedure
1. Run `scripts/collect-diff.sh <base>` to get commits and changed files.
2. Group changes by area (api/, ui/, infra/, tests/).
3. Fill in `references/template.md` with:
   - **Summary** (1–3 sentences, user-facing impact)
   - **Changes** (bulleted, grouped by area)
   - **Test plan** (what was verified, what to verify on review)
   - **Linked issue** (if provided)
4. Present the draft to the user before any `gh pr` call.

## Validation
- Summary mentions the user-facing effect, not just the implementation.
- No bullet repeats the commit list verbatim.
- Test plan is concrete (commands, URLs, or steps), not "tested locally".

## Failure modes
- Branch has 50+ commits: ask the user to confirm scope before drafting.
- No clear user-facing change (refactor): make that explicit in the summary
  rather than inventing one.
```

**Install it:**

```sh
npx skills add github.com/your-org/skills --skill pr-description-writer
```

That's the whole loop. Notice that the `description` says exactly when to activate, the body has predictable sections, and the heavy work (collecting the diff, the template format) lives in supporting files that load only when the skill runs.

---

## Using the `npx skills` CLI

### Install

```sh
# GitHub shorthand
npx skills add vercel-labs/agent-skills

# Full GitHub URL
npx skills add https://github.com/vercel-labs/agent-skills

# Direct path to one skill inside a repo
npx skills add https://github.com/vercel-labs/agent-skills/tree/main/skills/web-design-guidelines

# GitLab or any git URL
npx skills add https://gitlab.com/org/repo
npx skills add git@github.com:vercel-labs/agent-skills.git

# Local path
npx skills add ./my-local-skills
```

### Common flags

```sh
# Install globally (available across all projects, for the current user)
npx skills add -g vercel-labs/agent-skills

# Project-scope install is the default — no flag needed; the skill is written
# into the current repo so your team gets it on checkout.

# List skills in a repo without installing
npx skills add vercel-labs/agent-skills --list

# Install only specific skills by name
npx skills add vercel-labs/agent-skills --skill frontend-design --skill skill-creator

# Install everything in a repo
npx skills add vercel-labs/agent-skills --all

# Target a specific agent (Claude Code, Cursor, Codex, OpenCode, ...)
npx skills add vercel-labs/agent-skills --agent claude-code

# Copy files instead of symlinking (default is symlink — survives `git update`
# without re-running install; use --copy in environments without symlink support)
npx skills add vercel-labs/agent-skills --copy

# Non-interactive, for CI
npx skills add vercel-labs/agent-skills --skill frontend-design -g -y
```

### Ongoing management

```sh
npx skills list                        # what's installed
npx skills list -g                     # global skills only
npx skills list -a claude-code         # filter by agent

npx skills find                        # interactive search
npx skills find typescript             # keyword search

npx skills check                       # check for updates
npx skills update                      # update everything
npx skills update <skill-name>         # update one

npx skills remove                      # interactive
npx skills remove web-design-guidelines

npx skills init                        # scaffold a new skill
npx skills init my-skill
```

### Where skills land on disk

Project scope installs into the agent's config directory in your repo — for example `.claude/skills/` for Claude Code, `.agents/skills/` for several others. Commit that directory so your team picks up skills on checkout. Global installs go under your home directory in the same per-agent location.

---

## Publishing and discoverability

There is no special publish command. To "publish":

1. Put your skill(s) in a git repo.
2. Make the repo accessible (public on GitHub/GitLab, or share the URL internally).
3. Share the `npx skills add <repo>` command.

The CLI discovers skills in standard repo locations — `skills/`, `skills/.curated/`, `skills/.experimental/`, and agent-specific directories. If none of those exist, it falls back to a recursive search.

Whether your skill is listed on skills.sh depends on the platform's discovery process; install activity flows through anonymous telemetry by default, and you can opt out by setting `DISABLE_TELEMETRY=1`. If you want guaranteed listing or curation, check the current skills.sh submission process — don't assume installs alone are enough.

What a public skill repo should include:

- One or more skill folders with `SKILL.md`
- A `README.md` with what the skills do and how to install them
- A clear license
- Safety notes for any `scripts/` (what they run, what they touch)

---

## Security: treat skills like code

Skills can ship executable scripts. A malicious or careless skill can run anything your shell can run.

- Read `SKILL.md` and every file in `scripts/` before installing.
- Prefer skills from organizations or maintainers you already trust.
- Review diffs on `npx skills update` — the CLI shows what changed; don't auto-accept.
- For team installs, pin to a specific commit or tag in the repo URL when stability matters more than freshness.

---

## FAQ

**My skill isn't activating — why?**
The `description` is the trigger. Re-read it as if you were the agent: does it name the keywords a user would actually type? If it says "Helps with deployments" instead of "Use when the user mentions deploy, Vercel, preview URL, or rollback", the agent has nothing to match against.

**Where are skills installed on disk?**
Per-agent inside your project (e.g. `.claude/skills/`, `.agents/skills/`) for project installs, or under your home directory in the same per-agent path for `-g` global installs. Run `npx skills list` to see exact paths.

**Can I use a skill in CI?**
Yes — pass `-y` to skip prompts:
```sh
npx skills add your-org/skills --skill ci-helper -g -y
```

**How do I version a skill?**
Use `metadata.version` with semver and tag your repo. Bump on breaking changes to the body or supporting files; `npx skills check` will surface available updates.

**Skill vs MCP server — which do I use?**
A skill is markdown + optional scripts loaded into the agent's context. An MCP server is a long-running process the agent calls over a protocol. Start with a skill. Reach for MCP only when you need strict typed payloads, persistent state, or tight control over external API calls.

**Can I keep a skill private to my team?**
Yes — host it in a private git repo. The `npx skills add` command works with any git URL your shell can clone (SSH keys, GitHub access tokens, etc.).

---

## Next steps

- Run `npx skills init my-first-skill` to scaffold a template.
- Browse [skills.sh](https://skills.sh) and [vercel-labs/agent-skills](https://github.com/vercel-labs/agent-skills) for real examples to read and remix.
- Copy the PR description writer above into a local folder and `npx skills add ./pr-description-writer` to try the full loop end to end.
