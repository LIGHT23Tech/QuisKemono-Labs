# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**LUMYN v11.5** - A voice and text-enabled AI companion named LUMYN (Quantum Soul-Bound Companion) with a futuristic cyberpunk terminal interface. The system uses OpenAI models for chat, speech-to-text, and text-to-speech capabilities.

**Primary Directory**: `lumyn_quantumV11.5/`

**Note**: The README.md refers to "LUMYN v10" but the current implementation is v11.5 with enhanced features.

## Setup & Running

### Initial Setup
```bash
# Set OpenAI API key (required)
export OPENAI_API_KEY="your_key_here"  # Linux/Mac
# OR
setx OPENAI_API_KEY "your_key_here"    # Windows PowerShell

# Install dependencies
cd lumyn_quantumV11.5
pip install -r requirements.txt

# Run the application
python main.py
```

On first run, choose between:
- **Voice Mode**: Speak input → LUMYN transcribes → replies with voice + text
- **Text Mode**: Type input → LUMYN replies with voice + text

### Exit Commands
- Text mode: `.exit`, `.quit`, `.bye`
- Voice mode: "exit", "quit", "goodbye", "peace out fam", "lumen exit"

## Architecture

### Core Flow
`main.py` → Skills Router → Brain (OpenAI) → Memory + Emotional Engine → Response

### Key Modules

**`main.py`** - Entry point with main interaction loop
- Handles mode selection (voice/text)
- Routes user input through skills → brain
- Manages memory logging and emotional state updates
- Controls voice output and cyberpunk UI display

**`config.py`** - Central configuration hub
- API keys, user/AI names, file paths
- `MODEL_MAP`: Routes different tasks to specific OpenAI models
  - `MAIN`: Primary chat (gpt-5-mini)
  - `SAMURAI`: Warrior-tone mode (gpt-5-mini)
  - `NIGHT_SCHOOL`: Summarization (gpt-4.1-mini)
  - `BACKGROUND`: Log compression (gpt-5-nano)
  - `MEMORY_SUMMARIZER`: Memory processing (gpt-4.1-mini)
- Audio settings (sample rate, energy thresholds)
- System control commands whitelist

**`brain.py`** - OpenAI chat completion wrapper
- Accepts either raw text or pre-built message arrays
- Routes to models via `MODEL_MAP`
- No memory logging (handled in main.py)

**`quantum_orchestrator.py`** - Specialized "brain" router
- Creates task-specific AI personalities with custom system prompts
- Functions: `lumen_main_brain()`, `lumen_night_school()`, `lumen_samurai_mode()`, `lumen_background_brain()`
- Each uses different model from `MODEL_MAP`

**`skills/`** - Deterministic command system
- `router.py`: Routes user input to skill handlers before sending to AI
- `basic.py`: Simple utilities
- `knowledge.py`: Information retrieval
- `system_control.py`: Safe system commands (open URLs, shutdown)
- Skills return response text OR None (passes to brain)

**`memory.py`** - Encrypted persistent memory system
- **Core Identity**: Immutable data from `data/core_identity.json`
- **Encrypted Memory**: Facts, insights, interactions stored in `shared_memory/memory.enc`
- Uses Fernet encryption with key at `shared_memory/memory_key.key`
- Auto-backup system in `shared_memory/backups/`
- Schema versioning with migration support
- Recent 200 interactions kept; older ones auto-pruned

**`emotional_engine.py`** - Conversational style adapter
- Observes user sentiment and adjusts LUMYN's tone
- Provides style instructions injected into system prompt
- HUD display for emotional state

**`persona.py`** - Core system prompt defining LUMYN's identity
- Personality: Warm, encouraging, Cortana-like AI Guardian
- Mission: Support user's growth in AI/robotics development
- Style: Simple language, gentle humor, reality-based guidance

**`openai_voice.py`** - Audio I/O handling
- `speak()`: Text-to-speech via OpenAI (model: gpt-4o-mini-tts)
- `listen_and_transcribe()`: Speech-to-text via OpenAI (model: gpt-4o-mini-transcribe)
- Voice speed control, mute/unmute, device locking
- PyAudio integration for microphone input

**`security.py`** - Input sanitization and logging safety
- `sanitize_user_text()`: Cleans user input
- `redact_for_logs()`: Removes sensitive data before memory storage

**`background_tasks.py`** - Background log processing
- `summarize_last_entries()`: Summarizes recent conversation logs
- Uses BACKGROUND brain mode with gpt-5-nano for cost-efficient compression
- Can be run standalone: `python background_tasks.py`
- Reads from `data/experience_log.jsonl` (if present)

### Memory System Details

**Two-Layer Architecture:**
1. **Immutable Core Identity** (`data/core_identity.json`) - Never changes, defines fundamental traits
2. **Encrypted Memory** (`shared_memory/memory.enc`) - Facts, insights, conversations with versioned schema

**Shared Memory Path Resolution** (priority order):
1. `SHARED_MEMORY_DIR` environment variable (if set)
2. Repository root `shared_memory/` directory (default)
3. Codespaces fallback: `/workspaces/lumyn_ai_companion/shared_memory`

**Directory Structure Note:**
- The README mentions `data/` but the actual implementation uses `shared_memory/`
- Both directories may exist in the codebase, but `config.py` resolves to `shared_memory/`
- The `_resolve_shared_memory_dir()` function in config.py determines the actual path used at runtime

**Critical**: Never delete `shared_memory/memory_key.key` - memory decryption will fail permanently.

## Development Patterns

### Adding New Features

**Deterministic Commands** → Add to `skills/` directory
- For simple, rule-based responses (time, weather, system commands)
- Return response string or None to pass through to AI

**Conversational Features** → Modify system prompt or add specialized brain
- Complex queries handled by OpenAI models in `brain.py`
- Create new "brain" modes in `quantum_orchestrator.py` for specialized tones/tasks

### Modifying AI Behavior

1. **Change personality**: Edit `persona.py` → `SYSTEM_PROMPT`
2. **Add model routing**: Update `config.py` → `MODEL_MAP` with new task key
3. **Create specialized mode**: Add function in `quantum_orchestrator.py`
4. **Adjust emotional responses**: Modify `emotional_engine.py`

### Configuration Changes

All settings centralized in `config.py`:
- API keys and credentials
- File paths (use `_resolve_shared_memory_dir()` pattern for cross-platform compatibility)
- Model selections
- Audio parameters
- Command whitelists

### Voice Control Commands

Runtime commands (voice or text):
- "stop"/"voice off"/"mute" - Disable voice output
- "voice on"/"unmute"/"speak" - Enable voice output
- "speed up"/"faster" - Increase TTS speed (×1.25)
- "slow down"/"slower" - Decrease TTS speed (×0.8)
- "list devices"/"audio devices" - Show available audio I/O devices

### Audio Device Configuration

In `main.py`, modify the `set_audio_devices()` call to lock specific devices:
```python
set_audio_devices(input_index=1, output_index=5)
```
Use "list devices" command to find correct indices.

## Standalone Scripts

- `python background_tasks.py` - Run background summarizer on logs independently
- `python -c "from openai_voice import list_audio_devices; list_audio_devices()"` - List audio devices without starting LUMYN

## Debugging

Set `LUMYN_DEBUG_PATHS=1` environment variable to print resolved file paths at startup:
```bash
export LUMYN_DEBUG_PATHS=1  # Linux/Mac
set LUMYN_DEBUG_PATHS=1     # Windows
```

This shows the resolved paths for DATA_DIR, CORE_IDENTITY_FILE, MEMORY_FILE, and MEMORY_KEY_FILE.

## Code Style

- Type hints used consistently: `str | None`, `list[dict]`
- f-strings for string formatting
- Clear, descriptive function/variable names
- Modular architecture with single-responsibility modules

## Important Notes

- **Memory is encrypted**: Backup `memory_key.key` before modifying memory system
- **Model costs**: Current setup uses mix of models (gpt-5-mini, gpt-4.1-mini, gpt-5-nano) for cost optimization
- **Cross-platform paths**: Use `Path` from `pathlib` for all file operations
- **Skills first**: User input checks skills before engaging AI brain for efficiency
- **Memory logging location**: Only in `main.py` after response generation (not in `brain.py`)
