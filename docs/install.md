# Install

## Quick install (recommended)

```bash
curl -fsSL https://jaaicode.org/install.sh | sh
```

Works on macOS (Intel + Apple Silicon) and Linux without an existing
Python, Homebrew, or Xcode toolchain. The installer brings its own
Python runtime if needed and installs jaaicode into an isolated venv
at `~/.jaaicode/runtime/`.

After install:

```bash
jaaicode
```

Follow the first-run wizard to pick a model and key.

## From PyPI (Python ≥ 3.12 already installed)

```bash
pipx install jaaicode
```

or

```bash
pip install --user jaaicode
```

## Uninstall

```bash
rm -rf ~/.jaaicode
rm  ~/.local/bin/jaaicode
```

## First run

The wizard asks three things:

1. **Provider** — Ollama (local), OpenAI, Anthropic, Google, or any
   OpenAI-compatible base URL (LiteLLM, OpenRouter, …).
2. **Model** — defaulted from the provider; change with `/model` later.
3. **API key** — stored in your OS keychain, never written to disk in
   plaintext.

That's it. Type a question, watch it stream.
