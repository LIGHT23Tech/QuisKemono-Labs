# Project Overview

This is `lumyn_quantum` v11.5, a voice and text-enabled AI assistant named LUMYN. It is described as a "Quantum Soul-Bound Companion." The application runs in a terminal and presents a futuristic, cyberpunk-style interface with ASCII art and bordered output.

**Core Technologies:**
*   **Language:** Python
*   **AI Models:** OpenAI is used for speech-to-text, text-to-speech, and chat completion. The specific models are defined in `config.py` and include:
    *   `gpt-5-mini` (primary chat)
    *   `gpt-4.1-mini` (summarization, memory)
    *   `gpt-5-nano` (background tasks)
    *   `gpt-4o-mini-transcribe` (speech-to-text)
    *   `gpt-4o-mini-tts` (text-to-speech)
*   **Libraries:** Key libraries include `openai`, `pyaudio` for voice input, and `cryptography` for Fernet-based memory encryption.

**Architecture:**
The application has a modular architecture with a clear flow: User Input → Skills Router → Brain (OpenAI) → Memory + Emotional Engine → Response

**Key Modules:**

*   **`main.py`**: The main entry point containing the primary user interaction loop. Handles mode selection (voice/text), routes input through skills then brain, manages memory logging (the ONLY place interactions are logged), updates emotional state, and controls voice output with cyberpunk UI.

*   **`config.py`**: A central configuration file for managing API keys, user/AI names, file paths, and model names. Contains `MODEL_MAP` which routes different tasks to specific models:
    *   `MAIN`: Primary chat (gpt-5-mini)
    *   `SAMURAI`: Warrior-tone mode (gpt-5-mini)
    *   `NIGHT_SCHOOL`: Summarization (gpt-4.1-mini)
    *   `BACKGROUND`: Log compression (gpt-5-nano)
    *   `MEMORY_SUMMARIZER`: Memory processing (gpt-4.1-mini)

*   **`brain.py`**: A wrapper for the OpenAI chat completion API. Accepts either raw text or pre-built message arrays (with system/user roles). Routes to models via `MODEL_MAP`. Does NOT log memory (that's handled in main.py).

*   **`quantum_orchestrator.py`**: A routing system to create specialized "brains" for different tasks by combining unique system prompts with specific models from `MODEL_MAP`. Functions include:
    *   `lumen_main_brain()` - Standard conversational mode
    *   `lumen_night_school()` - Summarization and knowledge structuring
    *   `lumen_samurai_mode()` - Calm, disciplined warrior guidance
    *   `lumen_background_brain()` - Log compression and organization

*   **`skills/`**: A skill-based system for deterministic commands. The `skills/router.py` attempts to match user input to a programmatic skill (like controlling the system or fetching knowledge) BEFORE passing the prompt to the main AI brain. Skills return response text OR None (passes to brain). Contains:
    *   `basic.py` - Simple utilities
    *   `knowledge.py` - Information retrieval
    *   `system_control.py` - Safe system commands (open URLs, shutdown)

*   **`memory.py`**: Manages a persistent, encrypted long-term memory using a two-layer architecture:
    1. **Core Identity** (immutable): Loaded from `data/core_identity.json` - fundamental traits that never change
    2. **Encrypted Memory**: Facts, insights, and interactions stored in `shared_memory/memory.enc` with Fernet encryption
    *   Uses `shared_memory/memory_key.key` for encryption/decryption
    *   Auto-backup system in `shared_memory/backups/`
    *   Schema versioning with migration support
    *   Keeps recent 200 interactions; auto-prunes older ones

*   **`emotional_engine.py`**: Adjusts the AI's conversational style based on user sentiment and interaction patterns. Provides style instructions that get injected into the system prompt. Shows emotional state in a HUD display.

*   **`persona.py`**: Contains the core system prompt (`SYSTEM_PROMPT`) that defines LUMYN's identity as a warm, encouraging, Cortana-like AI Guardian focused on supporting the user's growth in AI/robotics development.

*   **`openai_voice.py`**: Handles audio I/O operations:
    *   `speak()` - Text-to-speech via OpenAI
    *   `listen_and_transcribe()` - Speech-to-text via OpenAI
    *   Voice speed control, mute/unmute, audio device locking
    *   PyAudio integration for microphone input

*   **`security.py`**: Input sanitization and logging safety with `sanitize_user_text()` and `redact_for_logs()`.

# Building and Running

1.  **Set up API Key:**
    The application requires an OpenAI API key. It should be set as an environment variable named `OPENAI_API_KEY`.

    *Example (PowerShell):*
    ```powershell
    setx OPENAI_API_KEY "your_key_here"
    ```
    *Example (Bash):*
    ```bash
    export OPENAI_API_KEY="your_key_here"
    ```

2.  **Install Dependencies:**
    Install the required Python packages from `requirements.txt`.
    ```bash
    cd lumyn_quantumV11.5
    pip install -r requirements.txt
    ```

3.  **Run the Application:**
    Execute the `main.py` script from within the `lumyn_quantumV11.5` directory.
    ```bash
    python main.py
    ```
    On first run, you will be prompted to choose between:
    *   **Voice Mode** - Speak input → LUMYN transcribes → replies with voice + text
    *   **Text Mode** - Type input → LUMYN replies with voice + text

4.  **Exit Commands:**
    *   **Text Mode:** `.exit`, `.quit`, `.bye`
    *   **Voice Mode:** "exit", "quit", "goodbye", "bye", "peace out", "peace out fam", "lumen exit"

5.  **Voice Control Commands** (available in both modes):
    *   "stop"/"voice off"/"mute" - Disable voice output (text only)
    *   "voice on"/"unmute"/"speak" - Enable voice output
    *   "speed up"/"faster" - Increase TTS speed (×1.25)
    *   "slow down"/"slower" - Decrease TTS speed (×0.8)
    *   "list devices"/"audio devices" - Show available audio I/O devices

# Development Conventions

## Configuration Management
*   **Centralized Settings:** All important settings, paths, and model choices are centralized in `config.py`. To modify behavior, start by looking there.
*   **Shared Memory Path:** Uses `_resolve_shared_memory_dir()` pattern for cross-platform compatibility. Priority order:
    1. `SHARED_MEMORY_DIR` environment variable (explicit override)
    2. Repository root `shared_memory/` directory (default)
    3. Codespaces fallback: `/workspaces/lumyn_ai_companion/shared_memory`
*   **Audio Device Locking:** In `main.py`, modify `set_audio_devices(input_index=1, output_index=5)` to lock specific devices. Use "list devices" command to find correct indices.

## Memory System
*   **Critical:** The application's memory is stored in the `shared_memory/` directory. The encryption key (`memory_key.key`) is REQUIRED to decrypt the memory file (`memory.enc`). **Do not delete this key** - memory decryption will fail permanently.
*   **Two-Layer Architecture:**
    1. **Core Identity** (`data/core_identity.json`) - Immutable, fundamental traits
    2. **Encrypted Memory** (`shared_memory/memory.enc`) - Facts, insights, conversations
*   **Backups:** Auto-backups stored in `shared_memory/backups/` with timestamps
*   **Schema Versioning:** Memory format uses versioned schema with automatic migrations

## Skills vs. Brain
*   **Skills First:** User input is checked against skills BEFORE engaging the AI brain for efficiency
*   **Skills** (`skills/` directory) - Use for simple, deterministic, rule-based commands:
    *   Examples: Time queries, system commands, preset responses
    *   Return response string OR None to pass through to AI
*   **Brain** (`brain.py`) - Use for complex or conversational queries:
    *   Examples: Open-ended questions, creative tasks, context-aware responses
    *   Powered by OpenAI models via `MODEL_MAP`

## Adding New Features

**For Deterministic Commands:**
1. Add handler function to appropriate file in `skills/` directory
2. Skills return response text OR None (passes to brain)
3. Router in `skills/router.py` will automatically check new skills

**For Conversational Features:**
1. Modify system prompt in `persona.py` for personality changes
2. Create new "brain" mode in `quantum_orchestrator.py` for specialized tones/tasks
3. Add new model routing key to `MODEL_MAP` in `config.py`

**For Modifying AI Behavior:**
1. **Personality:** Edit `persona.py` → `SYSTEM_PROMPT`
2. **Model Selection:** Update `config.py` → `MODEL_MAP`
3. **Specialized Modes:** Add function in `quantum_orchestrator.py`
4. **Emotional Responses:** Modify `emotional_engine.py`

## Modular Brains Pattern
*   The `quantum_orchestrator.py` provides a pattern for creating specialized "brains" by combining:
    *   Unique system prompt (defines task and tone)
    *   Specific model from `MODEL_MAP` (optimizes for cost/capability)
*   Examples: NIGHT_SCHOOL (summarization), SAMURAI (warrior guidance), BACKGROUND (log compression)

## Code Style
*   Type hints used consistently: `str | None`, `list[dict]`
*   f-strings for string formatting
*   Clear, descriptive function and variable names
*   Single-responsibility modules
*   Cross-platform paths using `Path` from `pathlib`

## Important Notes
*   **Memory Logging Location:** Only happens in `main.py` after response generation (NOT in `brain.py`)
*   **Model Cost Optimization:** Uses mix of models (gpt-5-mini, gpt-4.1-mini, gpt-5-nano) for cost efficiency
*   **Safe System Commands:** Whitelist in `config.py` → `ALLOWED_SYSTEM_COMMANDS`
