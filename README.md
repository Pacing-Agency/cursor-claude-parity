<p align="center">
  <img src="docs/banner.svg" alt="Pacing: cursor-claude-parity. One rule set for Cursor and Claude Code." width="100%">
</p>

# cursor-claude-parity

Write your agent rules, agents, skills, MCP servers and hooks once under `.cursor/`, and generate the Claude Code side from them with one standard-library Python script.

[![Tests](https://github.com/Pacing-Agency/cursor-claude-parity/actions/workflows/tests.yml/badge.svg)](https://github.com/Pacing-Agency/cursor-claude-parity/actions/workflows/tests.yml)
[![Licence: MIT](https://img.shields.io/badge/licence-MIT-orange.svg)](LICENSE)
[![Python 3.9+](https://img.shields.io/badge/python-3.9%2B-blue.svg)](https://www.python.org/)
[![Dependencies: none](https://img.shields.io/badge/dependencies-none-brightgreen.svg)](cursor_claude_parity.py)

## Who it's for

Teams and solo developers who use Cursor and Claude Code on the same repo and are tired of keeping two sets of instructions in step.

## What problem it solves

- **Rules drift.** Edit a rule in `.cursor/rules/` and forget `CLAUDE.md`, and the two tools now follow different guardrails.
- **Claude Code does not read `.mdc` imports.** It only inlines `@` imports of `.md` files, so an always-on Cursor rule is silently missing unless it is copied. The script copies it and `--check` catches stale copies.
- **MCP and hook config differ in small ways.** `${env:VAR}` against `${VAR}`, event names, model slugs. The script converts them so you do not hand-edit JSON.

## Quick start

```bash
git clone https://github.com/Pacing-Agency/cursor-claude-parity.git
cp cursor-claude-parity/cursor_claude_parity.py /path/to/your-repo/
cd /path/to/your-repo

python cursor_claude_parity.py .            # generate the Claude Code files
python cursor_claude_parity.py . --check    # drift check, exit 1 on drift
```

Try it on the bundled example first:

```bash
cp -r cursor-claude-parity/examples/tiny-repo /tmp/demo
python cursor-claude-parity/cursor_claude_parity.py /tmp/demo
```

Run `--check` in CI or a pre-commit hook to stop the two sides drifting.

## How it works

| Source | Generated |
|---|---|
| `.cursor/mcp.json` | `.mcp.json` (`${env:VAR}` becomes `${VAR}`, `envFile` dropped, `type` set on URL servers) |
| `.cursor/rules/*.mdc` | `CLAUDE.md` index block, `.claude/rules/*.md` (copies of always-on rules, path-scoped pointers for glob rules) |
| `.cursor/agents/*.md` | `.claude/agents/*.md` (thin pointers, Cursor model slugs dropped) |
| `.cursor/skills/*/SKILL.md` | `.claude/skills/*/SKILL.md` (pointers) |
| `.cursor/hooks.json` | `.claude/settings.json` (event names converted) |

Everything it writes carries a marker comment. It never overwrites a hand-written file, and it only touches the block between the `claude-parity` markers in `CLAUDE.md`, so your own prose stays. It never reads `.env`.

Exit codes: 0 fine, 1 drift or a conflict with a hand-written file, 2 bad arguments, 3 no rules found (pass `--allow-no-rules` to override).

### Example

Before, `.cursor/mcp.json`:

```json
"docs": { "url": "https://mcp.example.com/mcp",
          "headers": { "Authorization": "Bearer ${env:DOCS_TOKEN}" } }
```

After, `.mcp.json`:

```json
"docs": { "url": "https://mcp.example.com/mcp",
          "headers": { "Authorization": "Bearer ${DOCS_TOKEN}" },
          "type": "http" }
```

Before, `.cursor/agents/reviewer.md` has `model: claude-sonnet-4` and a description ending "Use proactively after a PR is opened." After, `.claude/agents/reviewer.md` has the model slug gone, a lane note leading the description, and a body that points back to the Cursor file:

```
Canonical instructions: `.cursor/agents/reviewer.md`. Read that file and follow it completely.
```

The always-on rule becomes `.claude/rules/house-style.md`. The glob rule becomes a path-scoped pointer that Claude Code loads when it touches matching files. `CLAUDE.md` lists the always-on rules and adds a "read when" table for rules that have a description but neither `alwaysApply` nor globs.

### Options

All optional. Put them in `.cursor-claude-parity.json` at the repo root, pass `--config FILE`, or use flags.

| Config key | Flag | What it does |
|---|---|---|
| `env_defaults` | `--env-default NAME=VALUE` | Public default for `${env:NAME}`, written as `${NAME:-VALUE}`. Never use for secrets. |
| `lane_note` | `--lane-note TEXT` | Sentence prefixed to every agent description, so Claude Code does not start an agent you want Cursor to own. Also removes "use proactively". |
| `shared_agent_prefixes` | `--shared-agent-prefix P` | Agents whose file name starts with `P` skip the lane note. |
| `claude_flag_hooks` | `--flag-hook TEXT` | Hook commands containing `TEXT` get `--claude` appended, for scripts that behave differently under Claude Code. |
| `skip_hooks` | `--skip-hook TEXT` | Hook commands containing `TEXT` are not ported. |
| `hook_matchers` | none | `[{"contains": "x.cjs", "matcher": "Edit\|Write"}]` sets a PreToolUse matcher. |
| `claude_template` | `--claude-template PATH` | Starting `CLAUDE.md` for repos that have none. |
| `gitignore_lines` | none | Lines ensured in `.gitignore`. Default: `.claude/settings.local.json`, `CLAUDE.local.md`. |

### Hooks

Cursor hook events are converted to Claude Code names: `sessionStart` to `SessionStart`, `sessionEnd` to `SessionEnd`, `preToolUse` to `PreToolUse`, `postToolUse` to `PostToolUse`, `beforeSubmitPrompt` to `UserPromptSubmit`. Any other event is not ported, and the script prints a `note:` line naming it. If `.claude/settings.json` already exists and was not generated by this script, hooks are not ported at all and a note says so.

## Limits

- **One-way.** It generates Claude Code files from `.cursor/`. It does not read Claude Code config back into Cursor.
- **Not on PyPI.** There is no pip package: copy the single file into your repo or onto your path.
- **Some things do not transfer:** Cursor marketplace plugins, canvases, and Cursor OAuth sessions (remote MCP servers need re-authorising in Claude Code with `/mcp`). Cursor model slugs in agent files are dropped, and only `sonnet`, `opus`, `haiku` and `inherit` are kept.
- **Hooks are partial.** Only the five events above are converted.
- **Conventions can change.** Claude Code's file formats are not ours. Run `--check` after upgrading either tool.

## Tests

```bash
python -m unittest discover -s tests
```

Pairs with [pacing-ai-starter](https://github.com/Pacing-Agency/pacing-ai-starter), which carries this generator plus template rules for `.cursor/rules/` and a secret-guard pre-commit hook.

## Contributing

Issues and pull requests are welcome. Keep it standard library only, add a test for any behaviour change, and make sure `python -m unittest discover -s tests` passes.

## Licence

[MIT](LICENSE).

---

Built by [Pacing](https://pacing.agency), a London marketing and build agency. MIT. More on this project: [pacing.agency/built/cursor-claude-parity](https://pacing.agency/built/cursor-claude-parity).
