# Features

## Interfaces

- **CLI** — Rich-rendered REPL with streaming, history, command queue.
- **Textual TUI** — fullscreen terminal UI with input bar, scrollable
  output, collapsible tool cards, modals, footer status.
- **Web UI** — browser-served chat with tool cards and live streaming.
- **Daemon mode** — long-running server with Unix-domain-socket
  transport so multiple clients share a session.

## Model & routing

- Local Ollama and any OpenAI-compatible endpoint.
- Auto model selector — pick a tier (cheap / standard / heavy) from
  the prompt and active context.
- Auto cost router with per-session token + dollar tracking and
  budgets.
- Vision-capable model gating (warns and drops images on text-only
  models instead of 400-ing).
- Mid-turn model escalation when the loop guard detects the small
  model is stuck.

## Agent loop

- Iterative, bounded tool-calling loop (XML and native tool-call
  formats both supported).
- Auto-extending iteration cap — only warns on a real hard cap.
- Loop guard — observes assistant text, tool calls, and file writes;
  stops on detected loops, repeat-write storms, or no-progress
  patterns.
- Pause (Esc) keeps the conversation consistent: interrupt flag
  stops the loop at the next iteration boundary instead of
  mid-call.
- Queued user submissions fold into the running turn at iteration
  boundaries — clarifications typed mid-task arrive on the next
  iteration as one merged user message.
- Reflection memory on tool errors; deferred LSP diagnostics flushed
  per-iteration after writes.

## Tools

- **File ops** — read, write, multi-write, edit, apply diff, batch
  regex edit.
- **Search** — grep, glob, symbol search, find-callers, list-dir,
  language-server queries (definition / references / hover /
  diagnostics), repomap (PageRank-ranked symbol map).
- **Refactoring** — project-wide rename, extract-function, inline.
- **Shell** — rlimits / sandbox backends, syntax-highlighted output.
- **Test runner** — structured failure parsing.
- **Git** — status / diff / log / blame / commit / pickaxe search.
- **Web** — fetch, search, headless browser (text + screenshot).
- **Notebook** — edit / run Jupyter cells.
- **Code analysis** — dependency graph, dead-code, complexity,
  duplication, architecture map, type coverage, migration plan,
  diff impact, quality score.
- **Memory & doctrine** — persistent project facts plus a
  `JAAICODE.md` doctrine file the agent maintains in cooperation
  with the user.
- **Sub-agents** — general / explore / planner / test-runner /
  reviewer spawned as autonomous helpers.
- **Cron** — schedule prompts to run on a cadence.

## Sessions, skills, plugins

- Named sessions with replay, transcript export, undo of file edits.
- Skill library — auto-activated short instructions that match a
  prompt.
- Plugin system for extending the tool surface.
- Slash commands: `/help`, `/queue`, `/model`, `/sessions`,
  `/review`, `/diff`, `/skills`, `/store`, `/workplan`, `/replay`,
  `/mcp`, …

## Quality & safety

- Pre-commit hook enforcing CHANGELOG updates for user-visible
  files via per-repo `.changelog-watch` patterns.
- Confirmation modals for destructive ops; YOLO mode bypasses with
  a clear footer indicator.
- Token budget guard, hard-cap iteration guard, loop guard — three
  independent kill switches.

## Modes

- `chat` (default) and `code` (full tool access).
- `plan` mode — read-only tools only, designed for exploring a
  codebase and writing a plan before editing.
- Mode personas: `architect`, `ask`, `debug`, `review`, etc.
