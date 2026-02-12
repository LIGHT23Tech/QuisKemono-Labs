# Repository Guidelines

## Project Structure & Module Organization
This repository centers on `lumyn_quantumV11.5/`, a Python voice/text AI companion app.
- Core runtime: `lumyn_quantumV11.5/main.py`
- Model and behavior modules: `brain.py`, `memory.py`, `emotional_engine.py`, `security.py`, `persona.py`, `quantum_orchestrator.py`
- Skill routing and handlers: `lumyn_quantumV11.5/skills/`
- Configuration and environment-dependent paths: `lumyn_quantumV11.5/config.py`
- Persistent data: `lumyn_quantumV11.5/data/` and `lumyn_quantumV11.5/shared_memory/`
- Project notes for other agents: `CLAUDE.md`, `GEMINI.md`

## Build, Test, and Development Commands
Run commands from `lumyn_quantumV11.5/` unless noted.
- `python -m pip install -r requirements.txt`: install dependencies.
- `python main.py`: start the assistant UI (voice/text mode selection).
- `OPENAI_API_KEY=... python main.py`: run with explicit API key in shell.
- `python -m py_compile *.py skills/*.py`: quick syntax check before PR.

## Coding Style & Naming Conventions
- Follow PEP 8 with 4-space indentation and readable, small functions.
- Use `snake_case` for variables/functions, `PascalCase` for classes, `UPPER_CASE` for constants (see `config.py`).
- Keep type hints where practical (existing code uses `str | None`, `list[dict]`, etc.).
- Prefer explicit imports and avoid side effects outside startup/config code.
- Keep comments brief and only where intent is not obvious.

## Testing Guidelines
There is currently no formal automated test suite in this repo.
- Add tests under a new `tests/` directory using `pytest` for new logic-heavy modules.
- Name files as `test_<module>.py` and tests as `test_<behavior>()`.
- Minimum for changes: run `python -m py_compile *.py skills/*.py` and do a manual smoke run with `python main.py`.

## Commit & Pull Request Guidelines
Git history is currently minimal (`Initial commit: LUMYN v11.5 AI Companion`) and uses descriptive, sentence-style subjects.
- Prefer concise imperative commits, e.g. `fix: sanitize exit phrase handling`.
- Keep commits focused (one logical change per commit).
- PRs should include: purpose, key files changed, manual test evidence, and any config/env updates.
- Link related issues and include terminal screenshots/log snippets for UI or voice-flow changes.

## Security & Configuration Tips
- Never commit secrets; use environment variables for `OPENAI_API_KEY`.
- Treat `shared_memory/`, `data/`, and encryption key artifacts as sensitive runtime data.
- Review command-execution paths in `security.py` and `skills/system_control.py` carefully before expanding capabilities.
