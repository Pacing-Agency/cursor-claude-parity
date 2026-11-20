# cursor-claude-parity

> One standard-library Python script that generates the Claude Code config from a Cursor rule set.

## What this is

`cursor_claude_parity.py` reads `.cursor/` (rules, agents, skills, MCP servers, hooks) in a target repo and writes the Claude Code side: `.mcp.json`, the marked block in `CLAUDE.md`, `.claude/rules/`, `.claude/agents/`, `.claude/skills/` and `.claude/settings.json`. It is one-way, and `--check` reports drift. Public repo, MIT licence. The README is the user guide.

## Stack

- Python 3.9 or newer, standard library only. Keep it that way.
- Tests: `unittest`.

## Local setup

```bash
python -m unittest discover -s tests     # run the tests
python cursor_claude_parity.py examples/tiny-repo --check
```

Try the generator on a copy of `examples/tiny-repo`, not in place.

## Deploy

None. It is a single file that users copy into their own repo. There is no package to publish.

## Where things live

- `cursor_claude_parity.py` - the whole generator
- `tests/test_parity.py` - tests
- `examples/tiny-repo/` - a small demo repo
- `docs/` - banner images for the README
- `.github/workflows/tests.yml` - CI

## Gotchas

- Never read or print `.env` values; the script must not read `.env` at all.
- The script never overwrites a hand-written file and only edits the block between the `claude-parity` markers in `CLAUDE.md`.
- Add a test for any behaviour change. British English, no em dashes.

## Monitoring

Nothing runs on a schedule and there is no service. The only workflow is `tests.yml`, which runs the unit tests on every push and pull request (Python 3.9 and 3.13). Nothing else watches this repo.
