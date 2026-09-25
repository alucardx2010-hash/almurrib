# المعرب (Almurrib)

An **open-source Arabic game localization ecosystem** — not just a
translator. The long-term goal is a modular system that can take a real
game, understand how its text is stored, extract the translatable content,
translate it (locally first, cloud optional with BYO key), validate the
result, process Arabic correctly (reshaping, BiDi, fonts), and put the
translation back into the game.

> **Status: multi-engine platform (Phase 3).** Detection → extraction →
> glossary + translation (cloud BYO-key, offline Argos, or local models) →
> Arabic QA → per-engine patch (Ren'Py, RPG Maker MV/MZ, Unity) with
> originals untouched. See [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md),
> [docs/ENGINE_SUPPORT.md](docs/ENGINE_SUPPORT.md) (honest capability
> matrix), [docs/PROVIDERS.md](docs/PROVIDERS.md) and
> [docs/PHASE3_ARCHITECTURE.md](docs/PHASE3_ARCHITECTURE.md).

## Why an ecosystem, not a tool

Inspired by GalTransl, LunaTranslator, VNTextPatch and Textractor: the
valuable thing is a modular ecosystem where extraction, translation, QA,
Arabic processing, reinjection, and runtime support work together — and
can survive beyond the original author. Mature open-source building
blocks (UnityPy, repak, llama.cpp, CTranslate2, Argos Translate, ...)
are integrated through adapters, never forked into the core.

## What works today

```text
Game → detect (Ren'Py / RPG Maker / Unity) → extract → glossary + translate
       (cloud, offline Argos, or local llama.cpp) → Arabic QA
       → engine patch: strings.rpy / translated JSON / rebuilt assets
       (originals untouched)
```

Validated end-to-end on Ren'Py's official demo "The Question" (77 entries)
and the RPG Maker MV fixture (32 entries); Unity on synthetic layout.

## Providers

Pick from 20+ definitions (OpenRouter, TokenRouter, Agent Router, OpenAI,
Gemini, DeepSeek, Groq, Together, Fireworks, DeepInfra, Cerebras, SambaNova,
Mistral, xAI, Cohere native, Hugging Face, Ollama / llama.cpp / vLLM / NIM
local, …). The GUI offers a provider selector, **Test Connection**,
**Fetch Models** (searchable model dropdown, manual id fallback) and a
force-retranslation checkbox. See [docs/PROVIDERS.md](docs/PROVIDERS.md) for
the verified compatibility matrix.

## Quick start

```powershell
# Python 3.11+ required. Runtime is near-stdlib (Unicode libs only);
# heavy stacks are optional extras: .[unity] (UnityPy), .[offline] (Argos).
python -m venv .venv
.\.venv\Scripts\python -m pip install -e ".[dev]"

# Desktop GUI (Tkinter frontend: provider selector, model discovery, logs):
.\.venv\Scripts\almurrib-gui

# Offline walkthrough on the bundled tiny fixture (no API key needed):
.\.venv\Scripts\almurrib detect  fixtures\renpy_tiny
.\.venv\Scripts\almurrib extract fixtures\renpy_tiny --db demo.db --json out\entries.json
.\.venv\Scripts\almurrib inspect --db demo.db
.\.venv\Scripts\almurrib db      --db demo.db

# Full localization with a real AI provider (Bring Your Own Key):
$env:ALMURRIB_API_KEY  = "..."                              # never committed
$env:ALMURRIB_BASE_URL = "https://api.openai.com/v1"        # or any registry provider
$env:ALMURRIB_MODEL    = "gpt-4o-mini"
.\.venv\Scripts\almurrib localize <game_dir> --db demo.db --out patch\ `
  --provider openai --model gpt-4o-mini
# → patch\game\tl\arabic\strings.rpy, ready to drop into a copy of the game

# Repeat safely: re-extract marks vanished lines obsolete (history kept),
# re-translate reuses cache/TM with zero API calls, --force starts fresh:
.\.venv\Scripts\almurrib translate --db demo.db --game-dir fixtures\renpy_tiny --force

# Run the tests (offline; live API test is opt-in):
.\.venv\Scripts\python -m pytest
```

Copy `.env.example` to `.env` for persistent local configuration.
When double-clicking the packaged EXE, `.env` / the default database /
output live next to the executable (see [docs/GUI.md](docs/GUI.md)).

### Build the Windows app

```powershell
.\.venv\Scripts\python build_exe.py     # → dist\Almurrib.exe
```

See [docs/GUI.md](docs/GUI.md) for the GUI and executable details.

## Layout

```text
src/almurrib/
  core/            normalized model (+provenance), pipeline, cache, providers,
                   placeholders, config, translation stage, workflow,
                   shared error reporting (engine-agnostic)
  arabic/          Arabic layer: normalize, masking, reshape, BiDi, wrap,
                   fonts, structured QA (canonical logical text always kept)
  engine_adapters/ engine-specific code (renpy/: parser, adapter, reinjection)
  providers/       registry + discovery + generic OpenAI transport + Cohere
                   native adapter + fake (tests) + factory
  storage/         SQLite database (migrations) + repository + persistent cache
  cli/             command-line interface (force/scope/provider parity w/ GUI)
  gui/             Tkinter frontend (provider/model discovery, logs, options)
tests/
  unit/            model, storage, cache, pipeline, parser, placeholders,
                   providers, discovery, registry, reporting, ...
  integration/     CLI workflow, repeated-run stability, fixture round-trip,
                   real-game validation, GUI behavior
  live/            opt-in live API + discovery tests (ALMURRIB_RUN_LIVE_TESTS=1)
fixtures/
  renpy_tiny/      deterministic sample Ren'Py game
  the_question/    Ren'Py's official demo script (real-game validation)
docs/              architecture notes and phase status
```

## Principles

Open source first · local first · cloud optional (BYO key) · opt-in sharing
(open formats: TMX/XLIFF) · long-term maintainability · reuse existing open
source via adapters · our value is the integration layer.

## License

AGPL-3.0 — deliberately, to keep the ecosystem open. See
[LICENSE](LICENSE) and [THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md).

## Documentation

This section provides additional information about the project structure and local development workflow
