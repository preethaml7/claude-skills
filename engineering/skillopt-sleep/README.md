# SkillOpt-Sleep (vendored plugin)

This folder is a **verbatim copy** of the `skillopt_sleep` engine and the
Claude Code plugin surface from
[microsoft/SkillOpt](https://github.com/microsoft/SkillOpt)
(`skillopt_sleep/`, `plugins/claude-code/`, `plugins/run-sleep.sh`). No logic
was modified — only relocated so path resolution (`CLAUDE_PLUGIN_ROOT`-relative
lookups in `scripts/sleep.sh` / `scripts/run-sleep.sh`) resolves correctly at
this folder's location. Licensed under the [MIT License](LICENSE) ©
Microsoft Corporation.

One cosmetic wording change was made to satisfy this repo's `scripts/check_paths.py`
CI gate: `skills/skillopt-sleep/SKILL.md`'s frontmatter `description` said
"...consolidate validated CLAUDE.md/SKILL.md behind a held-out gate" — the
`CLAUDE.md/SKILL.md` substring reads as a broken relative path to that
linter's `[A-Za-z0-9_\-./]+/SKILL\.md` regex. Reworded to "CLAUDE.md and
SKILL.md"; no behavior or meaning changed. Carry this rewording forward on
re-vendor.

## What this plugin is

SkillOpt-Sleep gives a local Claude Code agent a nightly **sleep cycle**: it
reviews real past sessions in this repo, replays recurring tasks offline on
your own API budget, and consolidates what it learns into this repo's
`CLAUDE.md` memory and `SKILL.md` skills — but **only** through a held-out
validation gate, and **only** after you explicitly adopt the staged proposal.

It is the deployment-time companion to the (not vendored) `skillopt` training
package: SkillOpt trains a skill offline against a labeled benchmark;
SkillOpt-Sleep applies the same bounded-edit + held-out-gate discipline to
*actual usage of this repo* instead, so it needs no benchmark dataset.

```
harvest ~/.claude transcripts (read-only)
  → mine recurring tasks
  → replay offline
  → consolidate (reflect → bounded edit → GATE)
  → stage proposal (nothing live changes)
  → you review and run "adopt" (backs up first)
```

## Why this is a fit for a skills library with no test harness

This repo's [CLAUDE.md](../../CLAUDE.md) intentionally has no build system or
test framework, and skill `scripts/` are stdlib-only with no LLM calls, so the
full `microsoft/SkillOpt` training package (benchmark-driven, requires
labeled train/val/test data per task, needs `numpy`/`openai`/`azure-*`) was
**not** vendored — there is no natural ground-truth benchmark for something
like `finance/dcf-valuation` or `c-level-advisor/vpe-advisor`.

`skillopt_sleep`, by contrast, has **zero third-party dependencies** (stdlib
only), its default `mock` backend spends no API budget, and it mines its
"benchmark" from how the skills in *this* repo actually get used in real
sessions rather than a pre-labeled dataset. That matches the repo's
deterministic-first, portable-first philosophy far better than the training
package does.

## Use in this repo

```bash
# from the repo root:
engineering/skillopt-sleep/scripts/sleep.sh status                          # what's happened (read-only)
engineering/skillopt-sleep/scripts/sleep.sh dry-run  --project "$(pwd)"     # safe preview, stages nothing
engineering/skillopt-sleep/scripts/sleep.sh run      --project "$(pwd)"     # full cycle, stages a proposal
engineering/skillopt-sleep/scripts/sleep.sh adopt    --project "$(pwd)"     # apply staged proposal (backs up first)
```

Or, once the plugin is installed via Claude Code's plugin marketplace, use
the bundled `/skillopt-sleep [run|dry-run|status|adopt|harvest|schedule|unschedule]`
slash command (see `commands/skillopt-sleep.md` and `skills/skillopt-sleep/SKILL.md`).

Default backend is `mock` (deterministic, **no API spend** — safe to try
immediately). Add `--backend claude` to spend real budget replaying this
repo's own recurring tasks and get genuine lift on `CLAUDE.md` / a target
`SKILL.md`.

## Safety model (unchanged from upstream)

- Harvest is **read-only** over `~/.claude` session transcripts.
- Edits are proposed, gated against a held-out replay slice, and **staged**
  under `.skillopt-sleep/staging/<date>/` — nothing live is touched.
- `adopt` is explicit and backs up the prior file first (unless you opt into
  `--auto-adopt`).
- Per-night token/task budget caps; secrets are redacted from prompts.

## What was and wasn't vendored

| Vendored | Not vendored |
|---|---|
| `skillopt_sleep/` engine (stdlib-only) | `skillopt/` training package (needs `numpy`/`openai`/`azure-*` + labeled benchmarks) |
| `plugins/claude-code/skills\|hooks\|commands\|scripts/` | `plugins/codex/`, `plugins/copilot/`, `plugins/devin/`, `plugins/openclaw/` (other-agent plugin variants) |
| `plugins/run-sleep.sh` shared launcher | `skillopt_webui/` (optional Gradio dashboard) |
| `LICENSE` | `docs/`, `ckpt/`, `data/`, `index.html` (training-package docs/site/checkpoints) |

## Updating

Re-vendor from upstream when the plugin changes:

```bash
git clone --depth 1 https://github.com/microsoft/SkillOpt.git /tmp/skillopt-upstream
cp -r /tmp/skillopt-upstream/skillopt_sleep engineering/skillopt-sleep/skillopt_sleep
cp -r /tmp/skillopt-upstream/plugins/claude-code/skills/skillopt-sleep engineering/skillopt-sleep/skills/skillopt-sleep
cp -r /tmp/skillopt-upstream/plugins/claude-code/hooks engineering/skillopt-sleep/hooks
cp -r /tmp/skillopt-upstream/plugins/claude-code/commands engineering/skillopt-sleep/commands
cp /tmp/skillopt-upstream/plugins/claude-code/scripts/sleep.sh /tmp/skillopt-upstream/plugins/claude-code/scripts/install-cron.sh engineering/skillopt-sleep/scripts/
cp /tmp/skillopt-upstream/plugins/run-sleep.sh engineering/skillopt-sleep/scripts/run-sleep.sh
cp /tmp/skillopt-upstream/LICENSE engineering/skillopt-sleep/LICENSE
```
