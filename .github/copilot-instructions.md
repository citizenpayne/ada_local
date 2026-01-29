# Copilot / AI Agent Instructions for A.D.A (pocket_ai)

Short, actionable guidance for an AI coding agent to be productive in this repository.

Overview
- Purpose: A.D.A is a fully-local Windows-focused AI assistant. Key runtime pieces live under `core/` (backend logic) and `gui/` (PySide6 front-end). The project favors local-only tooling: a fine-tuned FunctionGemma router (`merged_model/`) and Ollama for LLM inference.

Quick facts
- Entry point: `main.py` launches the GUI app.
- Central config: `config.py` (models, Ollama URL, wake word, LOCAL_ROUTER_PATH).
- Local LLM: Ollama is used (see `core/llm.py`). Default model name is in `config.py` (`RESPONDER_MODEL`).
- Router: FunctionGemma router is a fine-tuned model under `merged_model/` and referenced by `core/router.py`.
- Function execution: `core/function_executor.py` dispatches actions (Kasa, calendar, web search, etc.).
- Voice: Wake-word + STT/TTS pipeline in `core/stt.py`, `core/tts.py`, and orchestrated by `core/voice_assistant.py`.

Developer workflows / useful commands
- Create a venv and install deps (Windows):
  - `python -m venv venv`
  - `venv\\Scripts\\activate`
  - `pip install -r requirements.txt`
- Ollama model steps (required for local LLM):
  - `ollama pull qwen3:1.7b` (or the model in `config.py`)
  - `ollama serve` to run the Ollama server
  - Verify: `ollama list`
- Run the app: `python main.py`
- Run tests: `pytest tests/`

Repository patterns & conventions (explicit)
- Local-first: features expect local services (Ollama, Piper TTS). Prefer local config values in `config.py`.
- Router-driven intents: Natural language input is first classified by the router model (`core/router.py`). When adding new actions, update the router training and `core/function_executor.py` handler mapping.
- Filesystem model path: Do not hard-code external model URLs; respect `LOCAL_ROUTER_PATH` and `RESPONDER_MODEL` in `config.py`.
- GUI <-> Core communication: GUI components (in `gui/`) invoke backend via `gui/handlers.py` which calls functions in `core/`. Follow existing handler patterns when adding new GUI actions.
- Kasa integration: `core/kasa_control.py` uses `python-kasa` discovery (UDP/9999). Device discovery is network-dependent—tests that touch Kasa are in `tests/` but may be marked for integration.

File examples to inspect when implementing changes
- Intent routing: `core/router.py`, `merged_model/` (model artifacts)
- Execution & side-effects: `core/function_executor.py` (see how functions return structured results)
- LLM calls: `core/llm.py` (Ollama HTTP API usage)
- Voice stack: `core/stt.py`, `core/voice_assistant.py`, `core/tts.py`
- GUI entry & handlers: `gui/app.py`, `gui/handlers.py`, `gui/tabs/*`

Testing & safety notes
- Unit tests: `tests/` — run with `pytest tests/`. Tests involving real Kasa devices or Ollama may be integration-only and require services running.
- When modifying runtime behavior (wake-word, models), update `config.py` and add tests that mock external services where feasible.

Editing guidance for AI agents
- Keep changes local and minimal: respect existing public functions and file boundaries in `core/` and `gui/`.
- If adding a new routed function:
  1. Add intent handler in `core/function_executor.py` following existing handlers.
  2. Add any helper logic under `core/` (new file if substantial).
  3. Update training data for the router model (training artifacts are external; note `merged_model/` is the fine-tuned result).
  4. Add a small unit test in `tests/` mocking external I/O.
- Prefer reading `config.py` for runtime parameters rather than hard-coding values.

What not to assume
- Do not assume cloud APIs or API keys are present—the project is intentionally local-first.
- Do not alter GUI frameworks: the UI uses PySide6; large UI refactors require careful testing.

If unsure, inspect these files first:
- `main.py` — app launch
- `config.py` — runtime configuration
- `core/router.py`, `core/function_executor.py`, `core/llm.py`
- `gui/app.py`, `gui/handlers.py`

If this file should include other project-specific conventions, tell me which areas feel incomplete.
