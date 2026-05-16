# claude-octopus-marketplace

Override marketplace pinning the `octo` Claude Code plugin to
[lestephen/claude-octopus](https://github.com/lestephen/claude-octopus) —
a fork of [nyldn/claude-octopus](https://github.com/nyldn/claude-octopus)
with local patches.

## Patches over upstream v9.38.0

1. **`feat(codex)`** — `OCTOPUS_CODEX_APPROVAL` env var (default `never`)
   injects `-c approval_policy=<value>` into every `codex exec` line so
   headless orchestration doesn't hang on global
   `approval_policy = "on-request"` from `~/.codex/config.toml`.
2. **`fix(commands)`** — `embrace.md`, `discover.md`, `doctor.md` no
   longer `cd "${HOME}/.claude-octopus/plugin"` before invoking
   `orchestrate.sh`. They use the absolute path so the user's project
   directory stays as `$PWD` (otherwise Gemini's auto-detected
   workspace gets pinned to the plugin install dir and writes are
   refused as "outside workspace").
3. **`feat(orchestrate)`** — `PROJECT_ROOT` falls back to `$OLDPWD` or
   `$OCTOPUS_PROJECT_DIR` when `$PWD` is under `/.claude/plugins/cache/`
   or `.claude-octopus/plugin`. Belt-and-suspenders for the `.md` fix.
4. **`feat(gemini)`** — `scripts/lib/dispatch.sh` appends
   `--include-directories ${PROJECT_ROOT}` to the gemini flag set in
   headless mode, guaranteeing project access regardless of cwd.
5. **`fix(commands)`** — `/octo:doctor` resolver no longer creates a
   self-referential `~/.claude-octopus/plugin` symlink when invoked from
   a working install. Now resolves canonical paths via `readlink -f`
   and skips `ln -sfn` if source and target already match.
6. **`fix(doctor)`** — `doctor_check_skills` accepts skill directories
   instead of only regular files. Eliminates 51 false-positive
   "Skill file missing" failures (skills are dirs containing
   `SKILL.md`, not single files).
7. **`fix(doctor)`** — `doctor_check_recurrence` no longer aborts
   silently. `((recent_failures++))` returns exit 1 when incremented
   from 0, which under `set -eo pipefail` killed `do_doctor` before
   any output was rendered. Same `|| true` guard already used on the
   symmetric counter at line 259.
8. **`fix(doctor)`** — `doctor_check_agents` no longer invokes a
   `claude agents list` subcommand that never existed (CC v2.1.139+
   `claude agents` opens the Agent View TUI for managing background
   sessions, unrelated to plugin-declared subagents). Replaced with
   two stable checks: `enabledPlugins` parse for `octo@*` entries
   and `claude plugin validate` schema check. Directly answers "will
   my agents load?" with correct interfaces.

## Install

```
/plugin marketplace add lestephen/claude-octopus-marketplace
/plugin disable octo@nyldn-plugins
/plugin install octo@lestephen-octo
```

## Update path

When upstream v9.38.1+ ships:

```bash
cd ~/source/claude-octopus
git fetch upstream
git rebase upstream/main lestephen-patches
git tag v9.38.1-lestephen.1
git push origin lestephen-patches v9.38.1-lestephen.1
```

Then bump the `version` and `ref` fields in `marketplace.json` here
and `/plugin update octo`.
