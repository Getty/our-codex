# our-codex

*From each `.claude/`, to each `AGENTS.md`.*

Bridge your [Claude Code](https://claude.ai/code) skills and agents to [OpenAI Codex](https://developers.openai.com/codex/cli/). Write your knowledge base once in `.claude/`, use it everywhere.

## The Problem

Claude Code stores context in `.claude/skills/` and `.claude/agents/`. Codex reads `AGENTS.md`. If you use both tools, you're maintaining two copies of the same knowledge.

## The Solution

`our-codex` reads your `.claude/` hierarchy — skills, agents, `CLAUDE.md` — strips the YAML frontmatter, and assembles everything into a single `AGENTS.md` that Codex understands. Then it launches Codex.

```
.claude/skills/my-skill/SKILL.md  ──┐
.claude/agents/my-agent.md         ──┼──▶  AGENTS.md  ──▶  codex
CLAUDE.md                          ──┘
```

## Install

```bash
git clone https://github.com/Getty/our-codex.git
ln our-codex/our-codex ~/bin/our-codex   # or anywhere in PATH
```

**Requirements:** Perl 5 (core modules only), `npx`, `@openai/codex`

## Usage

```bash
our-codex [base-dir] [+skill ...] [-r|--resume] [-- codex-args...]
```

| Argument | Description |
|----------|-------------|
| `base-dir` | How far up to look for `.claude/` directories. Default: CWD only. Use `..` for parent, `../..` for grandparent. |
| `+skill-name` | Pull an extra skill by name from anywhere in the hierarchy (including `~/.claude/`). Can be repeated. |
| `-r`, `--resume` | Resume the last Codex session in this directory. Regenerates `AGENTS.md` first. |
| `-- args` | Everything after `--` is forwarded to Codex. |

## Examples

```bash
cd ~/projects/my-app

our-codex                           # Only my-app/.claude/ + CLAUDE.md
our-codex ..                        # Include parent .claude/ too
our-codex +github-cli               # CWD + github-cli skill from ~/.claude/
our-codex .. +vast-ai-cli           # Parent chain + extra skill
our-codex -r                        # Resume last session
our-codex -- --full-auto            # Pass --full-auto to Codex
our-codex .. -- "fix the tests"     # One-shot prompt with parent context
```

## How It Works

1. **Collects** `.claude/skills/*/SKILL.md`, `.claude/agents/*.md`, and `CLAUDE.md` from the current directory and optionally parent directories
2. **Strips** YAML frontmatter from each file, keeps the content body
3. **Cleans** agent descriptions (removes `<example>` blocks that are too verbose for Codex)
4. **Writes** everything into one `AGENTS.md` in the current directory with a sources index
5. **Launches** Codex (or resumes the last session with `-r`)

### Hierarchy

Content is ordered parent-first, project-last. Codex gives priority to content that appears later in `AGENTS.md`, so project-specific instructions win over generic ones.

The `+skill-name` syntax searches the full hierarchy (from `~/.claude/` down to CWD) without including everything from those directories — you pick exactly what you need.

### Size Limit

Codex has a 32 KiB default limit for `AGENTS.md`. If you exceed it, `our-codex` warns you. Increase the limit in `~/.codex/config.toml`:

```toml
project_doc_max_bytes = 65536
```

## .gitignore

`AGENTS.md` is generated — don't track it:

```gitignore
AGENTS.md
```

`our-codex` warns you if it's missing from `.gitignore`.

## Claude Code Skill/Agent Format

If you're new to Claude Code skills and agents:

**Skills** live in `.claude/skills/<name>/SKILL.md` — passive reference knowledge (API docs, conventions, usage patterns) injected into context.

**Agents** live in `.claude/agents/<name>.md` — autonomous workers with a system prompt, model selection, and tool restrictions.

Both use YAML frontmatter that `our-codex` strips when building `AGENTS.md`. The content body is pure markdown, which Codex handles natively.

## License

This is free software; you can redistribute it and/or modify it under the same terms as Perl 5.
