# Benchmarks

Monthly benchmark reports comparing coding-agent CLIs across a fixed
suite of complexity tiers (from a 1-file LRU cache to an 11.5K-LOC
real OSS codebase). Each report covers:

- **Capability**: does the agent solve the task? Scored by an
  independent test suite the agent never sees.
- **Cost**: USD per task at current retail rates, computed from raw
  token counts emitted by each agent's own telemetry.
- **Reliability**: variance across multiple trials per configuration.

The entire matrix — workspace setup, agent invocations, scoring,
data aggregation, this very page — is **orchestrated end-to-end with
[jaaicode](https://github.com/tomz/jaaicode)**. The methodology is
fully reproducible; see any report's "Reproducing" section for the
exact command.

Author: Tom Zeng ([github.com/tomz](https://github.com/tomz)).
Source for the bench harness lives in the [`jaaicode` repo](https://github.com/tomz/jaaicode)
under `scripts/bench-agents-matrix.py`. Raw per-run data is published
alongside each report as `*.jsonl`.

## Reports

| Date | Title | Highlights |
|---|---|---|
| **2026-05-10** | [Coding Agents Benchmark — May 2026](2026-05-10-coding-agents.md) | First run. 5 agents × 4 models × 5 tiers = 90 benchmark runs. Codex+gpt-5.4 and Copilot (both models) sweep every tier reliably; Claude Code and Gemini fail xlarge. |

## Methodology at a glance

| Tier | Source | LOC | What it tests |
|---|---|---:|---|
| `easy`   | `bugfix-cache` | 270 | Single-file bug fixing with adversarial test |
| `medium` | `bugfix-tinydb` | 550 | Multi-module bug fixing with cross-module deps |
| `high`   | `refactor R-007` | 145 | Surgical multi-file refactor |
| `large`  | `refactor R-015` | 450 | 26-file mechanical rename |
| `xlarge` | real `click` OSS | 11,500 | Navigate a real codebase, find + fix 3 surgical bugs |

The `xlarge` tier is the discriminator — most agents tie on smaller
tiers; xlarge separates the field.

## Cadence

- **Monthly** — first run of each month, published as soon as data
  cooks. Reports stay in the nav indefinitely.
- **On-demand** — `scripts/run-monthly-bench.sh` in the
  [jaaicode repo](https://github.com/tomz/jaaicode/tree/main/scripts)
  reproduces the run and emits a publishable markdown file.
