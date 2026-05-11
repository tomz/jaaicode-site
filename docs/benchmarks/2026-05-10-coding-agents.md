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

**Reliable across all 5 tiers (passes both trials on every tier):**

| Agent + Model | Total spend across matrix | xlarge cost | Notes |
|---|---:|---:|---|
| 🥇 **Codex + gpt-5.4** | $0.77 | **$0.12** | Best xlarge cost. 93–97% prompt-cache hit rate via LiteLLM's `/v1/responses`. Hit 34/35 on medium (one extra test). |
| 🥇 **jaaicode + gpt-5.4** | **$0.49** | **$0.08** | Cheapest reliable sweeper. Sub-second wall on large; fast on xlarge. |
| 🥇 **jaaicode + opus** | $1.54 | $0.92 | Stable across all tiers; pricey opus pricing but consistent. |
| 🥇 **claude-code + opus** | $2.93 | $1.42 | Stable across all tiers. |
| 🥇 **claude-code + sonnet** | $0.30 | $0.06 | Very cheap; sweeps every tier reliably. |
| 🥇 **Copilot + sonnet-4.5** | ≈$0.16 | ≈$0.04 | Cheapest if you have GitHub Copilot subscription. Only 2 premium reqs per task. |
| 🥇 **Copilot + opus-4.7** | ≈$3.00 | ≈$0.60 | Same scores as sonnet, 15× more premium requests. |

**Not reliable on all 5 tiers**:

| Agent + Model | Sweep | Notes |
|---|:-:|---|
| jaaicode + sonnet | 4/5 | xlarge: 50/50 on one of two trials (6/50 on the other). Cheap when it lands ($0.22). |
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

⭐ = perfect score reliably across both trials. ⚠ = one trial succeeded, one failed.
✗ = both trials failed.

| Tier (max) | jaai + sonnet | jaai + opus | jaai + gpt-5.4 | cc + sonnet | cc + opus | codex + gpt-5.4 | copilot + sonnet | copilot + opus | gemini-2.5-pro |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| `easy` (56) | **51/56** ⭐ / $0.05 | **51/56** ⭐ / $0.08 | **51/56** ⭐ / $0.06 | **51/56** ⭐ / $0.05 | **51/56** ⭐ / $0.51 | **51/56** ⭐ / $0.26 | **51/56** ⭐ / ≈$0.04 | 50/56 / ≈$0.60 | 35±23/56 ⚠ / $0.00 |
| `medium` (35) | **34/35** ⭐ / $0.09 | 33/35 ⭐ / $0.14 | 33/35 ⭐ / $0.10 | 33/35 ⭐ / $0.07 | 33/35 ⭐ / $0.49 | **34/35** ⭐⭐ / $0.15 | 33/35 / ≈$0.00 | 33/35 / ≈$0.60 | 33/35 / $0.00 |
| `high` (50)    | **50/50** ⭐ / $0.11 | **50/50** ⭐ / $0.09 | **50/50** ⭐ / $0.09 | **50/50** ⭐ / $0.03 | **50/50** ⭐ / $0.59 | 50/50 ⭐ / $0.16 | 50/50 / ≈$0.04 | 50/50 / ≈$0.60 | 50/50 / $0.00 |
| `large` (50)   | **50/50** ⭐ / **$0.03** | **50/50** ⭐ / $0.07 | **50/50** ⭐ / $0.09 | **50/50** ⭐ / $0.07 | **50/50** ⭐ / $0.44 | 50/50 / $0.07 | 50/50 / ≈$0.04 | 50/50 / ≈$0.60 | 50/50 / $0.00 |
| `xlarge` (50)  | 28±22/50 ⚠ / $0.22 | **50/50** ⭐ / $0.92 | **50/50** ⭐ / **$0.08** | **50/50** ⭐ / **$0.06** | **50/50** ⭐ / $1.42 | **50/50** ⭐ / $0.12 | **50/50** ⭐ / ≈$0.04 | **50/50** ⭐ / ≈$0.60 | 0/50 ✗ / $0.00 |

### Capability sweep table

| Agent + Model | easy | medium | high | large | xlarge | Sweep |
|---|:-:|:-:|:-:|:-:|:-:|:-:|
| **codex + gpt-5.4** | ✓ | ✓✓ | ✓ | ✓ | ✓ | **5/5** |
| **copilot + sonnet** | ✓ | ✓ | ✓ | ✓ | ✓ | **5/5** |
| **copilot + opus** | ✓ | ✓ | ✓ | ✓ | ✓ | **5/5** |
| **jaaicode + opus** | ✓ | ✓ | ✓ | ✓ | ✓ | **5/5** |
| **jaaicode + gpt-5.4** | ✓ | ✓ | ✓ | ✓ | ✓ | **5/5** |
| **claude-code + opus** | ✓ | ✓ | ✓ | ✓ | ✓ | **5/5** |
| **claude-code + sonnet** | ✓ | ✓ | ✓ | ✓ | ✓ | **5/5** |
| jaaicode + sonnet | ✓ | ✓ | ✓ | ✓ | ⚠ | 4/5 |
| gemini + 2.5-pro | ⚠ | ✓ | ✓ | ✓ | ✗ | 3/5 |

**Reliable cross-tier sweepers: 7 of 9** (all four LiteLLM-routed agent/model combos
on opus & gpt-5.4, plus both Copilot configurations and codex+gpt-5.4).

## Why the xlarge tier matters

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

**jaaicode + opus / gpt-5.4 deliver both trials at 50/50 on xlarge**
with consistent wall times (225–435 s) at costs of $0.08–$0.92. Its
`--pipe --execute --yolo` flow against LiteLLM produces deterministic
results once the model has the context.

**Claude Code succeeds on xlarge across both models**, with sonnet at
$0.06 and opus at $1.42.

**Gemini still fails xlarge.** Reads ~50 files in 450s without
converging. Verbose "thinking" output dominates; no edits to bug
locations.

## Cost detail across configurations (real money, May 2026 prices)

| Configuration | easy | medium | high | large | xlarge | Total |
|---|---:|---:|---:|---:|---:|---:|
| **jaaicode + gpt-5.4** | $0.06 | $0.10 | $0.09 | $0.09 | **$0.08** | **$0.42** |
| copilot + sonnet (overage) | $0.04 | $0.00 | $0.04 | $0.04 | $0.04 | $0.16 |
| **claude-code + sonnet** | $0.05 | $0.07 | $0.03 | $0.07 | $0.06 | $0.28 |
| **jaaicode + sonnet** | $0.05 | $0.09 | $0.11 | $0.03 | $0.22 | $0.50 |
| codex + gpt-5.4 | $0.26 | $0.15 | $0.16 | $0.07 | $0.12 | $0.77 |
| jaaicode + opus | $0.08 | $0.14 | $0.09 | $0.07 | $0.92 | $1.30 |
| claude-code + opus | $0.51 | $0.49 | $0.59 | $0.44 | $1.42 | $3.45 |
| copilot + opus (overage) | $0.60 | $0.60 | $0.60 | $0.60 | $0.60 | $3.00 |
| gemini-2.5-pro¹ | n/a | n/a | n/a | n/a | n/a | n/a |

¹ Gemini's token counts come back as 0 through LiteLLM. With Google's $1.25/$10 retail and the wall times we measured, estimated total ≈ $1.50–$2.50.

**Costs are noisy** because trial-to-trial variance dominates the
matrix at n=2. The Total column averages out some of that. Magnitude
ordering is reliable; ratios within 50% of each other should be
treated as ties.

## Sweet-spot recommendations

| If you need... | Use |
|---|---|
| Cheapest reliable sweep | **jaaicode + gpt-5.4** ($0.42 across all five tiers) |
| Most transparent telemetry, fully cached | **Codex + gpt-5.4** ($0.77 total, sweeps every tier with prompt-cache reporting) |
| Cheapest, GitHub Copilot subscription | **Copilot + sonnet** (≈$0 within quota) |
| Cheap & fast on routine tiers (≤large) | **jaaicode + sonnet** ($0.28 across easy-large) |
| Multi-model flexibility | **jaaicode** (any LiteLLM-exposed model) |
| Avoid completely on xlarge | gemini |

## Caveats

- **n=2 per configuration**. Adequate to spot capability ties, but small for tight cost confidence intervals. Run n≥5 if making procurement decisions.
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
(100+ records — 45 real configurations × 2 trials + 10 skip records, with both reported and corrected costs).

## Per-tier walkthrough — every agent

The five tables below show the full data for one tier each. Best cost
per tier highlighted **bold**. ⚠ = capability variance.

### `easy` — kvcache, 1 file, 270 LOC

| Agent + Model | Score | Wall | Cost | Input tok | Output tok |
|---|---:|---:|---:|---:|---:|
| jaaicode + sonnet | 51.0/56 | 30 s | **$0.05** | 59,642 | 868 |
| jaaicode + opus | 51.0/56 | 20 s | $0.08 | 35,380 | 746 |
| jaaicode + gpt-5.4 | 51.0/56 | 18 s | $0.06 | 48,937 | 582 |
| claude-code + sonnet | 51.0/56 | 28 s | **$0.05** | 14,359 | 648 |
| claude-code + opus | 51.0/56 | 22 s | $0.51 | 29,917 | 872 |
| codex + gpt-5.4 | 51.0/56 | 175 s | $0.26 | 285,801 | 10,579 |
| copilot + sonnet | 51.0/56 | 145 s | ≈$0.04 | (n/a — subscription) | — |
| copilot + opus | 50.0/56 | 68 s | ≈$0.60 | (n/a) | — |
| gemini + 2.5-pro | 35±23/56 ⚠ | 112 s | n/a | (n/a — 0 via proxy) | — |

### `medium` — tinydb, 4 modules, 550 LOC

| Agent + Model | Score | Wall | Cost | Input tok | Output tok |
|---|---:|---:|---:|---:|---:|
| jaaicode + sonnet | 34/35 | 50 s | $0.09 | 127,084 | 1,482 |
| jaaicode + opus | 33.0/35 | 42 s | $0.14 | 71,414 | 2,014 |
| jaaicode + gpt-5.4 | 33.0/35 | 22 s | $0.10 | 102,218 | 1,052 |
| claude-code + sonnet | 33.0/35 | 15 s | **$0.07** | 21,582 | 323 |
| claude-code + opus | 33.0/35 | 20 s | $0.49 | 29,718 | 569 |
| codex + gpt-5.4 | **34.0/35** ⭐ | 80 s | $0.15 | 206,852 | 4,796 |
| copilot + sonnet | 33.0/35 | 385 s | ≈$0 | (n/a) | — |
| copilot + opus | 33.0/35 | 60 s | ≈$0.60 | (n/a) | — |
| gemini + 2.5-pro | 33.0/35 | 80 s | n/a | (n/a) | — |

### `high` — refactor R-007, merge v1/v2 hierarchies

| Agent + Model | Score | Wall | Cost | Input tok | Output tok |
|---|---:|---:|---:|---:|---:|
| jaaicode + sonnet | 50.0/50 | 55 s | $0.11 | 121,650 | 2,244 |
| jaaicode + opus | 50.0/50 | 20 s | $0.09 | 65,262 | 788 |
| jaaicode + gpt-5.4 | 50.0/50 | 25 s | $0.09 | 168,582 | 878 |
| claude-code + sonnet | 50.0/50 | 38 s | **$0.03** | 9,213 | 1,164 |
| claude-code + opus | 50.0/50 | 35 s | $0.59 | 31,788 | 1,506 |
| codex + gpt-5.4 | 50.0/50 | 98 s | $0.16 | 181,296 | 6,657 |
| copilot + sonnet | 50.0/50 | 108 s | ≈$0.04 | (n/a) | — |
| copilot + opus | 50.0/50 | 50 s | ≈$0.60 | (n/a) | — |
| gemini + 2.5-pro | 50.0/50 | 82 s | n/a | (n/a) | — |

### `large` — refactor R-015, 26-file rename

| Agent + Model | Score | Wall | Cost | Input tok | Output tok |
|---|---:|---:|---:|---:|---:|
| jaaicode + sonnet | 50.0/50 | 20 s | **$0.03** | 32,028 | 626 |
| jaaicode + opus | 50.0/50 | 20 s | $0.07 | 52,610 | 528 |
| jaaicode + gpt-5.4 | 50.0/50 | 28 s | $0.09 | 154,850 | 1,100 |
| claude-code + sonnet | 50.0/50 | 35 s | $0.07 | 19,288 | 989 |
| claude-code + opus | 50.0/50 | 28 s | $0.44 | 25,431 | 758 |
| codex + gpt-5.4 | 50.0/50 | 42 s | $0.07 | 70,590 | 2,818 |
| copilot + sonnet | 50.0/50 | 48 s | ≈$0.04 | (n/a) | — |
| copilot + opus | 50.0/50 | 30 s | ≈$0.60 | (n/a) | — |
| gemini + 2.5-pro | 50.0/50 | 48 s | n/a | (n/a) | — |

### `xlarge` — click 11.5K LOC, 3 seeded bugs

| Agent + Model | Trial 1 | Trial 2 | Wall (mean) | Cost (mean) | Input tok (mean) |
|---|:-:|:-:|---:|---:|---:|
| jaaicode + sonnet | 6/50 | **50/50** | 195 s | $0.22 | 362,004 |
| **jaaicode + opus** | **50/50** | **50/50** | 330 s | $0.92 | 648,839 |
| **jaaicode + gpt-5.4** | **50/50** | **50/50** | **75 s** | **$0.08** | 139,722 |
| **claude-code + sonnet** | **50/50** | **50/50** | 32 s | **$0.06** | 19,080 |
| **claude-code + opus** | **50/50** | **50/50** | 1105 s | $1.42 | 54,542 |
| **codex + gpt-5.4** | **50/50** | **50/50** | 65 s | $0.12 | 211,818 |
| **copilot + sonnet** | **50/50** | **50/50** | 255 s | ≈$0.04 | (n/a) |
| **copilot + opus** | **50/50** | **50/50** | 502 s | ≈$0.60 | (n/a) |
| gemini + 2.5-pro | 0/50 | 0/50 | 450 s | n/a | (n/a) |

This tier separates the field at the top vs. bottom: **7 of 9 (model,
agent) configurations cross both trials at 50/50**. Gemini is the
only configuration that doesn't fix any seeded bug.
