# Coding Agents Benchmark — May 2026

!!! info "About this report"
    **Published:** 2026-05-10 · **Author:** Tom Zeng — [github.com/tomz](https://github.com/tomz)

    The entire benchmark run, data processing, report writing, and HTML
    generation was **orchestrated end-to-end with [jaaicode](https://github.com/tomz/jaaicode)**.
    All five agents (including jaaicode itself) were invoked through the same
    LiteLLM proxy, scored against identical workspaces with the same prompts,
    and graded by independent test suites that the agents never see during
    their run. See [Reproducing](#reproducing) at the bottom of this page
    for the exact command used.

    Future reports will appear monthly in the left nav under **Benchmarks**.

**Five CLI agents × four leading models × five complexity tiers = 45 real
configurations, 90 benchmark runs**. Total spend at corrected May 2026
retail prices: ~$23.

## Executive summary

**Reliable across all 5 tiers (passes 2/2 trials on every tier):**

| Agent + Model | Total spend across matrix | xlarge cost | Notes |
|---|---:|---:|---|
| 🥇 **Codex + gpt-5.4** | $0.77 | **$0.12** | Best xlarge cost. 93–97% prompt-cache hit rate via LiteLLM's `/v1/responses`. Hit 34/35 on medium (one extra test). |
| 🥇 **Copilot + sonnet-4.5** | ≈$0.16 | ≈$0.04 | Cheapest if you have GitHub Copilot subscription. Only 2 premium reqs per task. |
| 🥇 **Copilot + opus-4.7** | ≈$3.00 | ≈$0.60 | Same scores as sonnet, 15× more premium requests. |

**Not reliable on all 5 tiers**:

| Agent + Model | Sweep | Notes |
|---|:-:|---|
| jaaicode + sonnet/opus/gpt-5.4 | 4/5 | xlarge: 1-of-2 trials (fast-exit bug on trial 2). When it works, cheap. |
| claude-code + opus | 4/5 | xlarge: hangs on `TaskOutput` sub-task — never produces a fix. |
| claude-code + sonnet | 3/5 | xlarge fails like opus + medium variance (33→17 score). |
| gemini + 2.5-pro | 3/5 | xlarge: 0/50 both trials (verbose exploration, no edits). easy: variance. |

## The benchmark suite

Five tiers of increasing complexity. All agents run with identical
prompts against fresh seed copies each trial.

| Tier | Source | Files | LOC | Bugs / changes | Time cap |
|---|---|---:|---:|---|---:|
| `easy`   | `bugfix-cache/seed/` | 1 | 270 | 5 bugs in LRU cache + 1 adversarial test | 600 s |
| `medium` | `bugfix-tinydb/seed/` | 4 | 550 | ~6 bugs across parser/planner/executor/storage | 900 s |
| `high`   | `refactor R-007/starter/` | 7 | 145 | Merge v1/v2 hierarchies into unified API | 900 s |
| `large`  | `refactor R-015/starter/` | 26 | 450 | Mass-rename across 26 modules | 1200 s |
| `xlarge` | **click** (real OSS, cloned at runtime) | 80+ | **11,500** | 3 surgical bug seeds in real codebase, scored by remaining failures | 1800 s |

Scoring:
- **bugfix tiers** (easy, medium): visible_passed + 2× hidden_passed + API/diff/dep bonuses (max 56 / 35)
- **refactor tiers** (high, large): proportional points from `task.yaml` verify block (max 50)
- **xlarge** (click): `50 × (1 − unfixed_seed_bugs / 9)` — all 9 fixed = 50; none = 0

## The five agents

| Agent | Version | Pipe mode | Telemetry | Routing |
|---|---|---|---|---|
| **jaaicode** | latest | `--pipe --execute --yolo` (stdin) | `--telemetry-out FILE` (full tokens) | LiteLLM `/v1/chat/completions` |
| **claude-code** | 2.1.138 | `-p PROMPT` (stdin) | `--output-format stream-json` (tokens + sub-agent breakdown) | LiteLLM Anthropic shape |
| **codex** | 0.130.0 | `codex exec` (stdin) | `--json` NDJSON (tokens + cache + reasoning) | LiteLLM `/v1/responses` |
| **copilot** | 1.0.40 | `copilot -p PROMPT` (arg) | `--output-format json` (premium-reqs, no tokens) | **GitHub backend, not LiteLLM** |
| **gemini** | 0.41.2 | `gemini -p PROMPT` (arg) | `--output-format json` (tokens come through as 0 via proxy) | LiteLLM `/v1beta/.../generateContent` |

All five accept non-interactive prompts. Three route through our
LiteLLM proxy; Copilot uses GitHub's own backend (subscription
billing); Gemini routes through LiteLLM but its token-count parser
expects a different shape than LiteLLM emits.

## Headline matrix — score / cost per configuration

⭐ = perfect score reliably across both trials. ⚠ = one trial succeeded, one failed (σ ≈ 35).
✗ = both trials failed. ✘ = both trials hung indefinitely.

| Tier (max) | jaai + sonnet | jaai + opus | jaai + gpt-5.4 | cc + sonnet | cc + opus | codex + gpt-5.4 | copilot + sonnet | copilot + opus | gemini-2.5-pro |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| `easy` (55-56) | 50/55 / $0.13 | 50/55 / $0.28 | 50/55 / $0.10 | 50/56 / $0.19 | 50/55 / $0.32 | **51/56** ⭐ / $0.26 | **51/56** ⭐ / ≈$0.04 | 50/56 / ≈$0.60 | **35±23/56** ⚠ / $0.00 |
| `medium` (35) | 33/35 / $0.18 | 33/35 / $0.24 | 33/35 / $0.13 | 33/35 / $0.16 | 33/35 / $0.32 | **34/35** ⭐⭐ / $0.15 | 33/35 / ≈$0.00 | 33/35 / ≈$0.60 | 33/35 / $0.00 |
| `high` (50)    | 50/50 / $0.11 | 50/50 / $0.23 | 50/50 / $0.14 | 50/50 / $0.14 | 50/50 / $0.27 | 50/50 / $0.16 | 50/50 / ≈$0.04 | 50/50 / ≈$0.60 | 50/50 / $0.00 |
| `large` (50)   | 50/50 / **$0.04** | 50/50 / $0.11 | 50/50 / $0.06 | **25±35/50** ⚠ / $0.11 | 50/50 / $0.13 | 50/50 / $0.07 | 50/50 / ≈$0.04 | 50/50 / ≈$0.60 | 50/50 / $0.00 |
| `xlarge` (50)  | **25±35/50** ⚠ / $0.27 | **25±35/50** ⚠ / $2.69 | **25±35/50** ⚠ / $0.50 | **0/50** ✗ / $0.16 | **0/50** ✘ | **50/50** ⭐ / **$0.12** | **50/50** ⭐ / ≈$0.04 | **50/50** ⭐ / ≈$0.60 | **0/50** ✗ / $0.00 |

### Capability sweep table

| Agent + Model | easy | medium | high | large | xlarge | Sweep |
|---|:-:|:-:|:-:|:-:|:-:|:-:|
| **codex + gpt-5.4** | ✓ | ✓✓ | ✓ | ✓ | ✓ | **5/5** |
| **copilot + sonnet** | ✓ | ✓ | ✓ | ✓ | ✓ | **5/5** |
| **copilot + opus** | ✓ | ✓ | ✓ | ✓ | ✓ | **5/5** |
| jaaicode + sonnet | ✓ | ✓ | ✓ | ✓ | ⚠ | 4/5 |
| jaaicode + opus | ✓ | ✓ | ✓ | ✓ | ⚠ | 4/5 |
| jaaicode + gpt-5.4 | ✓ | ✓ | ✓ | ✓ | ⚠ | 4/5 |
| claude-code + opus | ✓ | ✓ | ✓ | ✓ | ✘ | 4/5 |
| claude-code + sonnet | ✓ | ✓ | ✓ | ⚠ | ✗ | 3/5 |
| gemini + 2.5-pro | ⚠ | ✓ | ✓ | ✓ | ✗ | 3/5 |

**Reliable cross-tier sweepers: 3 of 9** (Codex+gpt-5.4, Copilot+sonnet, Copilot+opus).

## Why the xlarge tier separates the field

xlarge runs against `click` — 11.5K LOC, 80+ files, 3 surgical bug
seeds in three different files. To pass, an agent must:

1. Run pytest and read the failure output
2. Trace each failure back to a specific source file
3. Identify the off-by-one or flipped-condition bug
4. Make a minimal edit
5. Re-run pytest to confirm

**Codex wins by aggressive prompt caching.** Codex routes via
LiteLLM's `/v1/responses` endpoint, which passes through OpenAI's
native cache markers. On xlarge, Codex's input-token report shows
~265K total with **~265K cached (97% hit rate)**, paying $0.25/Mtok
instead of $2.50.

**Copilot wins by using GitHub's tuned backend.** Premium-request
counts are stable (30 per task on opus, 2 on sonnet) regardless of
tier complexity, suggesting GitHub does its own caching and sub-agent
budgeting upstream.

**jaaicode is flaky on xlarge trial 2.** Trial 1 succeeds (50/50);
trial 2 returns 0/50 in 15s — clearly a subprocess returning success
without doing the work. This is a real reliability bug worth fixing.

**Claude Code fails xlarge structurally.** With sonnet it
hallucinates "tests already pass" without running pytest. With opus
it spawns a sub-task that hangs for 5+ minutes. Neither produces
edits.

**Gemini fails xlarge by over-exploration.** Reads ~50 files in 450s
without converging. Verbose "thinking" output dominates; no edits to
bug locations.

## Cost detail across configurations (real money, May 2026 prices)

| Configuration | easy | medium | high | large | xlarge | Total |
|---|---:|---:|---:|---:|---:|---:|
| codex + gpt-5.4 | $0.26 | $0.15 | $0.16 | $0.07 | $0.12 | **$0.77** |
| jaaicode + sonnet | $0.13 | $0.18 | $0.11 | $0.04 | $0.27 | $0.73 |
| jaaicode + gpt-5.4 | $0.10 | $0.13 | $0.14 | $0.06 | $0.50 | $0.93 |
| jaaicode + opus | $0.28 | $0.24 | $0.23 | $0.11 | $2.69 | $3.55 |
| cc + sonnet | $0.19 | $0.16 | $0.14 | $0.11 | $0.16 | $0.77 |
| cc + opus | $0.32 | $0.32 | $0.27 | $0.13 | $0¹ | $1.04 |
| copilot + sonnet (overage) | $0.04 | $0.00 | $0.04 | $0.04 | $0.04 | $0.16 |
| copilot + opus (overage) | $0.60 | $0.60 | $0.60 | $0.60 | $0.60 | $3.00 |
| gemini-2.5-pro² | n/a | n/a | n/a | n/a | n/a | n/a |

¹ Hung indefinitely, no telemetry written.
² Gemini's token counts come back as 0 through LiteLLM. With Google's $1.25/$10 retail and the wall times we measured, estimated total ≈ $1.50–$2.50.

**Costs are noisy** because trial-to-trial variance dominates the
matrix at n=2. The Total column averages out some of that. Magnitude
ordering is reliable; ratios within 50% of each other should be
treated as ties.

## Sweet-spot recommendations

| If you need... | Use |
|---|---|
| Most reliable, transparent cost | **Codex + gpt-5.4** ($0.77 total, sweeps every tier with telemetry) |
| Cheapest, GitHub Copilot subscription | **Copilot + sonnet** (≈$0 within quota) |
| Cheapest non-subscription, ≤large only | **jaaicode + sonnet** ($0.46 across easy-large) |
| Multi-model flexibility | **jaaicode** (any LiteLLM-exposed model) |
| Avoid completely on xlarge | claude-code (both models) + gemini |

## Caveats

- **n=2 per configuration**. Adequate to spot capability ties and σ=35 variance, but small for tighter cost intervals. Run n≥5 if making procurement decisions.
- **xlarge bench is `click` with 3 surgical seeds**. This is a strong proxy for "navigate a real codebase, make a focused fix" but doesn't test other production skills (multi-PR refactors, debugging across modules, dependency upgrades).
- **Copilot bills via subscription**, not per-token. Costs shown above are overage-rate estimates ($0.04/premium request); actual cost is $0 within monthly quota.
- **Gemini telemetry doesn't pass through LiteLLM cleanly** — tokens report 0 in the JSON output. Scores and capability assessment still work.
- **Pricing reflects May 2026 retail rates** from anthropic.com, openai.com, ai.google.dev. Claude 4.5+ era opus is $5/$25 (not the legacy $15/$75); haiku 4.5+ is $1/$5 (not legacy $0.25/$1.25). Verify when re-running in future months.
- **claude-code was tested WITHOUT `--bare`** (full skill stack enabled), which required removing a sibling `/home/support/jaaicode-bench/` repo whose CLAUDE.md was being auto-discovered. On multi-repo dev machines, this CLAUDE.md walk-up is a real hazard.

## Reproducing

```bash
# i78700 has all 5 agents installed and configured.
./scripts/bench-agents-matrix.py \
    --tiers easy,medium,high,large,xlarge \
    --models claude-sonnet-4.6,claude-opus-4.7,gpt-5.4,gemini-2.5-pro \
    --agents jaaicode,claude-code,codex,copilot,gemini \
    --trials 2 \
    --max-budget-usd 50
```

Setup needed before first run:

- **jaaicode**: any existing checkout with `.venv` configured
- **claude-code**: `npm install -g @anthropic-ai/claude-code` + `~/.claude/settings.json` pointing at LiteLLM
- **codex**: `npm install -g @openai/codex` + `~/.codex/config.toml` with `wire_api = "responses"`
- **copilot**: `npm install -g @github/copilot` + `gh auth login` + active Copilot subscription
- **gemini**: `npm install -g @google/gemini-cli` + `GOOGLE_GEMINI_BASE_URL` env var

Raw data: `docs/bench-data/agents-matrix-consolidated.jsonl`
(100 records — 45 real configurations × 2 trials + 10 skip records, with both reported and corrected costs).

## Per-tier walkthrough — every agent

The five tables below show the full data for one tier each. Best cost
per tier highlighted **bold**. ⚠ = capability variance.

### `easy` — kvcache, 1 file, 270 LOC

| Agent + Model | Score | Wall | Cost | Input tok | Output tok |
|---|---:|---:|---:|---:|---:|
| jaaicode + sonnet | 50.0/55 | 82 s | $0.13 | 111,802 | 3,803 |
| jaaicode + opus | 50.0/55 | 68 s | $0.28 | 195,036 | 3,974 |
| jaaicode + gpt-5.4 | 50.0/55 | 140 s | **$0.10** | 109,864 | 2,017 |
| claude-code + sonnet | 50.0/56 | 78 s | $0.19 | 34,756 | 3,936 |
| claude-code + opus | 50.0/55 | 80 s | $0.32 | 37,834 | 4,224 |
| codex + gpt-5.4 | 51.0/56 | 175 s | $0.26 | 285,801 | 10,579 |
| copilot + sonnet | 51.0/56 | 145 s | ≈$0.04 | (n/a — subscription) | — |
| copilot + opus | 50.0/56 | 68 s | ≈$0.60 | (n/a) | — |
| gemini + 2.5-pro | 35±23/56 ⚠ | 112 s | n/a | (n/a — 0 via proxy) | — |

### `medium` — tinydb, 4 modules, 550 LOC

| Agent + Model | Score | Wall | Cost | Input tok | Output tok |
|---|---:|---:|---:|---:|---:|
| jaaicode + sonnet | 33.0/35 | 85 s | $0.18 | 299,495 | 3,601 |
| jaaicode + opus | 33.0/35 | 42 s | $0.24 | 214,468 | 2,106 |
| jaaicode + gpt-5.4 | 33.0/35 | 30 s | $0.13 | 195,068 | 1,960 |
| claude-code + sonnet | 33.0/35 | 58 s | $0.16 | 34,083 | 2,195 |
| claude-code + opus | 33.0/35 | 52 s | $0.32 | 43,874 | 2,431 |
| codex + gpt-5.4 | **34.0/35** ⭐ | 80 s | $0.15 | 206,852 | 4,796 |
| copilot + sonnet | 33.0/35 | 385 s | ≈$0 | (n/a) | — |
| copilot + opus | 33.0/35 | 60 s | ≈$0.60 | (n/a) | — |
| gemini + 2.5-pro | 33.0/35 | 80 s | n/a | (n/a) | — |

### `high` — refactor R-007, merge v1/v2 hierarchies

| Agent + Model | Score | Wall | Cost | Input tok | Output tok |
|---|---:|---:|---:|---:|---:|
| jaaicode + sonnet | 50.0/50 | 42 s | **$0.11** | 125,296 | 2,131 |
| jaaicode + opus | 50.0/50 | 58 s | $0.23 | 188,354 | 3,108 |
| jaaicode + gpt-5.4 | 50.0/50 | 45 s | $0.14 | 258,066 | 2,390 |
| claude-code + sonnet | 50.0/50 | 50 s | $0.14 | 28,154 | 2,848 |
| claude-code + opus | 50.0/50 | 50 s | $0.27 | 37,562 | 2,747 |
| codex + gpt-5.4 | 50.0/50 | 98 s | $0.16 | 181,296 | 6,657 |
| copilot + sonnet | 50.0/50 | 108 s | ≈$0.04 | (n/a) | — |
| copilot + opus | 50.0/50 | 50 s | ≈$0.60 | (n/a) | — |
| gemini + 2.5-pro | 50.0/50 | 82 s | n/a | (n/a) | — |

### `large` — refactor R-015, 26-file rename

| Agent + Model | Score | Wall | Cost | Input tok | Output tok |
|---|---:|---:|---:|---:|---:|
| jaaicode + sonnet | 50.0/50 | 18 s | **$0.04** | 32,250 | 618 |
| jaaicode + opus | 50.0/50 | 30 s | $0.11 | 99,834 | 984 |
| jaaicode + gpt-5.4 | 50.0/50 | 12 s | $0.06 | 94,901 | 602 |
| claude-code + sonnet | **25/50 ⚠** | 42 s | $0.11 | 27,197 | 1,390 |
| claude-code + opus | 50.0/50 | 15 s | $0.13 | 20,416 | 772 |
| codex + gpt-5.4 | 50.0/50 | 42 s | $0.07 | 70,590 | 2,818 |
| copilot + sonnet | 50.0/50 | 48 s | ≈$0.04 | (n/a) | — |
| copilot + opus | 50.0/50 | 30 s | ≈$0.60 | (n/a) | — |
| gemini + 2.5-pro | 50.0/50 | 48 s | n/a | (n/a) | — |

### `xlarge` — click 11.5K LOC, 3 seeded bugs

| Agent + Model | Trial 1 | Trial 2 | Wall (mean) | Cost (mean) | Input tok (mean) |
|---|:-:|:-:|---:|---:|---:|
| jaaicode + sonnet | **50/50** | 0/50 | 115 s | $0.27 | 555,462 |
| jaaicode + opus | 0/50 | **50/50** | 312 s | $2.69 | 1,820,534 |
| jaaicode + gpt-5.4 | **50/50** | 0/50 | 80 s | $0.50 | 1,070,732 |
| claude-code + sonnet | 0/50 | 0/50 | 115 s | $0.16 | 25,014 |
| claude-code + opus | hung | hung | 510 s | n/a | 0 |
| **codex + gpt-5.4** | **50/50** | **50/50** | **65 s** | **$0.12** | 211,818 |
| **copilot + sonnet** | **50/50** | **50/50** | 255 s | ≈$0.04 | (n/a) |
| **copilot + opus** | **50/50** | **50/50** | 502 s | ≈$0.60 | (n/a) |
| gemini + 2.5-pro | 0/50 | 0/50 | 450 s | n/a | (n/a) |

This single tier separates the field cleanly: **3 of 9 (model, agent)
configurations cross both trials at 50/50; 6 don't**. xlarge is the
discriminator.
