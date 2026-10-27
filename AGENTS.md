# Agent notes

Agent skill for this tool: `devartslab-pnpm-fleet` (invoke by name; installed
at `~/.agents/skills/devartslab-pnpm-fleet/`). Do not copy it into this repo.

## Conventions

- `bin/pnpm-fleet` is a dependency-free bash CLI; keep it portable (macOS `sed -i ''`, node for JSON edits only).
- `fleet.conf` holds the fleet-pinned pnpm version; `pnpm-fleet pin` bumps it everywhere.
- `exceptions.list` is the only sanctioned escape hatch; audit flags anything else.
- This repo dogfoods its own policy: `.npmrc` and `packageManager` are managed by the tool.
