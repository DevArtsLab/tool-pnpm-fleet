# tool-pnpm-fleet

**One pinned toolchain. No duplicate versions. Every repo, always up to date.**

Fleet-wide pnpm governance for a directory of independent repos: a pinned
pnpm version enforced via `packageManager` + corepack, strict single-version
dependency trees per repo, security/major-version upgrade cadence, and an
explicit exception registry for edge cases.

## The policy

| Rule | Enforcement |
| ---- | ----------- |
| Same pnpm version everywhere | `packageManager` field + `manage-package-manager-versions` in `.npmrc` |
| Single dep version within a repo | `pnpm dedupe --check` in the CI gate |
| No foreign lockfiles or package managers | `pnpm-fleet check` fails on `package-lock.json`/`yarn.lock` |
| Fresh releases cool down before install | `minimum-release-age=1440` (24h) in `.npmrc` |
| Known vulnerabilities block | `pnpm audit --audit-level=high` in the CI gate |
| Edge cases allowed, never silently | `exceptions.list` registry |

Cross-repo dependency *uniformity* is a drift report, not a hard rule:
`check` tracks which repos lag, but nothing force-upgrades a deployed app
because another repo bumped first. Majors always require a green build.

## Requirements

- pnpm 10+ (the repo pins `12.15.0` via `fleet.conf`)
- node (for JSON edits) and bash; no other dependencies

## Usage

```bash
bin/pnpm-fleet check [DIR]   # audit every repo under DIR (default: sibling dirs)
bin/pnpm-fleet apply REPO    # adopt policy in one repo: writes .npmrc + packageManager pin
bin/pnpm-fleet pin 12.16.0   # re-pin the whole fleet to a new pnpm release
bin/pnpm-fleet audit [DIR]   # pnpm audit --audit-level=high across the fleet
bin/pnpm-fleet dupes REPO    # fail if REPO resolves duplicate dep versions
```

Adopt in a repo still on npm:

```bash
bin/pnpm-fleet apply ../my-repo
cd ../my-repo && pnpm import && rm package-lock.json && rm -rf node_modules && pnpm install
```

## Install / reuse elsewhere

Clone anywhere; the CLI is path-independent (`pnpm-fleet check` defaults to
scanning sibling dirs). Or run without cloning via:

```bash
pnpm dlx github:DevArtsLab/tool-pnpm-fleet check ~/projects
```

To use in another org: copy the repo, retarget `ROOT` scans to your
workspace dir, and update `exceptions.list` + `fleet.conf`.

## CI gate

Copy `ci/pnpm-gate.yml` into a repo's `.github/workflows/`. It runs
`pnpm install --frozen-lockfile`, `pnpm dedupe --check`, and
`pnpm audit --audit-level=high` on every push and PR.

## Files

| File | Purpose |
| ---- | ------- |
| `fleet.conf` | fleet-pinned pnpm version + policy knobs |
| `policy/npmrc` | shared `.npmrc` template applied to repos |
| `exceptions.list` | repos exempted from enforcement, with reasons |
| `ci/pnpm-gate.yml` | drop-in GitHub Actions gate |

## Upgrade cadence

- pnpm toolchain: re-pin on each minor release, or immediately on
  security-relevant releases: `pnpm-fleet pin <version>`
- Dependencies: audit on a routine schedule (`pnpm-fleet audit`), majors
  merge only after a green build; no blind automerge
- Exception reviews: re-check `exceptions.list` entries each quarter

## License

MIT - see [LICENSE](LICENSE).
