# Quick And Dirty

An Agent Skill for disposable implementation work where the desired outcome is:

- maximum implementation speed
- aggressive parallelization when safe
- correctness and edge-case coverage
- no known bugs after verification
- deliberately low concern for long-term maintainability, elegance, or code style

Use this for one-off patches to open source apps, forks, local tools, and
features that are unlikely to be maintained after they work.

Do **not** use it for production infrastructure, durable product foundations,
database migrations, public API contracts, security-sensitive code, or work
where maintainability matters.

## What This Skill Does

The skill instructs an AI coding agent to:

- lock a result contract before coding
- adapt ceremony to input size — trivial inputs get a one-line contract, long
  PRDs get triaged into a compressed critical path
- triage long PRDs and detailed plans into the fastest valid execution path
- create isolated worktrees only when the orchestration overhead is justified
- split independent work across subagents when ownership is clean
- prefer the shallowest working patch over clean architecture
- spend the saved time on tests, smoke checks, and adversarial edge-case review
- apply a defined shipping bar so verification does not loop indefinitely
- surface risky cuts to the user as a single confirmation, never silently
- avoid false success claims without fresh verification evidence

The skill itself has no runtime dependencies. It is a plain Agent Skills
directory with:

```text
SKILL.md
references/
evals/
```

## Install For AI Agents

If you are an AI agent given this README, install the skill before trying to use
it. Pick the target directory for the host you are running in.

### Cursor

Cursor discovers portable skills from `~/.agents/skills/*/SKILL.md` and
`~/.cursor/skills/*/SKILL.md`. Prefer the portable location:

```bash
mkdir -p "$HOME/.agents/skills"
git clone git@github.com:pedro-fresha/quick-and-dirty.git "$HOME/.agents/skills/quick-and-dirty"
```

Then use the skill by asking for quick-and-dirty implementation work, or invoke:

```text
/quick-and-dirty
```

### OpenCode

OpenCode discovers compatible skills from `~/.agents/skills/*/SKILL.md`,
`~/.config/opencode/skills/*/SKILL.md`, and compatible Claude skill directories.
Prefer the portable location:

```bash
mkdir -p "$HOME/.agents/skills"
git clone git@github.com:pedro-fresha/quick-and-dirty.git "$HOME/.agents/skills/quick-and-dirty"
```

If the OpenCode configuration blocks `.agents` skills, use the native OpenCode
location instead:

```bash
mkdir -p "$HOME/.config/opencode/skills"
git clone git@github.com:pedro-fresha/quick-and-dirty.git "$HOME/.config/opencode/skills/quick-and-dirty"
```

### Claude Code

Claude Code discovers skills from `~/.claude/skills/*/SKILL.md` and
project-local `.claude/skills/*/SKILL.md`. Install into the Claude Code user
skills directory:

```bash
mkdir -p "$HOME/.claude/skills"
git clone git@github.com:pedro-fresha/quick-and-dirty.git "$HOME/.claude/skills/quick-and-dirty"
```

If Claude Code is already running and the skill does not appear, restart the
session so the new top-level skill directory is discovered.

## Project-Local Install

Use a project-local install when the skill should apply only to one repository.

For Cursor or OpenCode:

```bash
mkdir -p .agents/skills
git clone git@github.com:pedro-fresha/quick-and-dirty.git .agents/skills/quick-and-dirty
```

For Claude Code:

```bash
mkdir -p .claude/skills
git clone git@github.com:pedro-fresha/quick-and-dirty.git .claude/skills/quick-and-dirty
```

## Update An Existing Install

If the target directory already exists and is a clone of this repo, update it
instead of cloning again:

```bash
git -C "$HOME/.agents/skills/quick-and-dirty" pull --ff-only
```

For Claude Code installs:

```bash
git -C "$HOME/.claude/skills/quick-and-dirty" pull --ff-only
```

For OpenCode native installs:

```bash
git -C "$HOME/.config/opencode/skills/quick-and-dirty" pull --ff-only
```

If the directory exists but is not a git clone, do not overwrite it silently.
Ask the user whether to move it aside or install somewhere else.

## Verify Installation

Run this after installing:

```bash
test -f "$HOME/.agents/skills/quick-and-dirty/SKILL.md" && echo "quick-and-dirty installed"
```

For Claude Code:

```bash
test -f "$HOME/.claude/skills/quick-and-dirty/SKILL.md" && echo "quick-and-dirty installed"
```

The skill frontmatter should contain:

```yaml
name: quick-and-dirty
```

## Agent Procedure

When using this skill:

1. Confirm the user really wants disposable, speed-first implementation.
2. Write a short result contract: required behavior, non-goals, edge cases, and
   existing behavior that must not break.
3. If the prompt includes a long PRD, multi-stage plan, or large issue, compress
   it first: extract non-negotiables, identify safe corners to cut, collapse
   unnecessary stages, and split independent tracks.
4. Use a branch or worktree when the repo is dirty, the current branch is
   `main`/`master`, or parallel subagents will edit code.
5. Split only independent work across agents. Keep shared contracts and
   integration work sequential.
6. Implement the shallowest working patch.
7. Run targeted tests, edge-case checks, and the broadest reasonable final
   verification.
8. Report exactly what changed, what was verified, and any known risks.

## Example Prompt

```text
Use quick-and-dirty for this. Patch this open source app so it supports exporting
the current view as CSV. I do not care about maintainability; I care that it
works, handles edge cases, and is verified.
```

## Repository Layout

```text
.
├── SKILL.md
├── references/
│   ├── evaluation.md
│   ├── long-spec-triage.md
│   ├── prompt-templates.md
│   ├── shipping-bar.md
│   └── worktree-parallelization.md
├── evals/
│   └── evals.json
├── LICENSE
└── README.md
```

## License

MIT — see `LICENSE`.
