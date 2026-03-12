# our-codex

Single-file Perl script that bridges Claude Code `.claude/` configuration to OpenAI Codex `AGENTS.md`.

## Architecture

One script (`our-codex`), no dependencies beyond Perl 5 core modules. The script:

1. Walks the directory hierarchy collecting `.claude/skills/*/SKILL.md`, `.claude/agents/*.md`, and `CLAUDE.md`
2. Parses YAML frontmatter (simple regex, no YAML module needed)
3. Assembles content into one `AGENTS.md`
4. Launches Codex via `npx`

## Key Design Decisions

- **Core modules only** — no CPAN dependencies, runs anywhere Perl is installed
- **Hardlink-friendly** — skills and agents are typically hardlinked across repos; this script reads them in-place
- **No config file** — all behavior controlled via CLI arguments
- **Frontmatter stripping** — Claude Code frontmatter (model, tools, color, etc.) is irrelevant to Codex and gets stripped
- **`<example>` block removal** — Claude Code agent descriptions contain `<example>` blocks for triggering; too verbose for Codex context window

## Testing

```bash
# Generate AGENTS.md without launching Codex (pipe mode, no TTY)
echo | perl our-codex

# Test with a specific project
cd /path/to/project && perl /path/to/our-codex
```

The script only launches Codex when STDIN is a TTY (`-t STDIN`). When piped, it writes `AGENTS.md` and exits.

## Code Style

- Standard Perl (`use strict; use warnings;`)
- Helper functions prefixed with `_`
- No OOP — straightforward procedural script
