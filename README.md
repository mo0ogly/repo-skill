# Claude Code Skills

Reusable slash commands for [Claude Code](https://claude.ai/claude-code).

## Skills

| Skill | Description |
|-------|-------------|
| [`/repo-cleanup`](commands/repo-cleanup.md) | Multi-language repo professionalization (audit, cleanup, lint, tests, CI, docs) |

## Installation

Symlink the `commands/` directory into your Claude Code config:

```bash
# Back up existing commands (if any)
mv ~/.claude/commands ~/.claude/commands.bak

# Symlink
ln -s /path/to/claude-skills/commands ~/.claude/commands
```

Or copy individual skills:

```bash
cp commands/repo-cleanup.md ~/.claude/commands/
```

## Usage

In Claude Code, invoke a skill with:

```
/repo-cleanup https://github.com/user/repo
/repo-cleanup /path/to/local/repo
```

## Supported Languages

The `/repo-cleanup` skill auto-detects and supports:

- **Python** — ruff, pytest, pyproject.toml, mypy
- **Go** — golangci-lint, go test, benchmarks, GoDoc
- **JavaScript/TypeScript** — eslint, prettier, jest/vitest
- **Rust** — clippy, cargo test

## License

MIT

## Author

mo0ogly@proton.me
