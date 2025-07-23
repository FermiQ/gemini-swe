# config.py

## Overview

The `config.py` file is responsible for managing all configuration settings, constants, and initial state for the Gemini Engineer application. It loads settings from an external `config.json` file, defines default values, gathers operating system information, and prepares the tools and system prompt for the AI model.

## Key Components

### Global State and OS Information
-   **`os_info: Dict[str, Any]`**: A dictionary holding detailed information about the operating system, Python version, and available shells (Bash, PowerShell, etc.).
-   **`base_dir: Path`**: The current working directory for file operations, initialized to `Path.cwd()`. Can be modified at runtime.
-   **`git_context: Dict[str, Any]`**: Stores Git-related state like whether Git is enabled and the current branch.
-   **`model_context: Dict[str, Any]`**: Stores AI model-related state, such as the current model being used (default or reasoner).
-   **`security_context: Dict[str, Any]`**: Holds security settings, like whether to require confirmation for shell commands.
-   **`logging_context: Dict[str, Any]`**: Manages logging state, including whether logging is enabled, the log directory, and current log file.

### Configuration Loading
-   **`load_config() -> Dict[str, Any]`**:
    -   **Description:** Loads configuration settings from a `config.json` file located in the same directory as `config.py`.
    -   **Returns:** A dictionary containing the loaded configuration. Returns an empty dictionary if the file is not found or is invalid.
-   **`config: Dict[str, Any]`**: The dictionary holding the loaded configuration from `config.json`.

### Constants
-   **Command Prefixes**:
    -   `ADD_COMMAND_PREFIX: str = "/add "`
    -   `COMMIT_COMMAND_PREFIX: str = "/git commit "`
    -   `GIT_BRANCH_COMMAND_PREFIX: str = "/git branch "`
-   **File Limits**: Constants like `MAX_FILES_IN_ADD_DIR`, `MAX_FILE_SIZE_IN_ADD_DIR`, `MAX_FILE_CONTENT_SIZE_CREATE`, `MAX_MULTIPLE_READ_SIZE`. These are loaded from `config.json` with default fallbacks.
-   **Fuzzy Matching Config**: `MIN_FUZZY_SCORE`, `MIN_EDIT_SCORE`.
-   **Conversation Config**: `MAX_HISTORY_MESSAGES`, `MAX_CONTEXT_FILES`, `ESTIMATED_MAX_TOKENS`, etc., for managing conversation context size and truncation.
-   **Model Config**: `DEFAULT_MODEL`, `REASONER_MODEL` names.
-   **Security Config**: `DEFAULT_SECURITY_CONTEXT` including flags like `require_powershell_confirmation`.
-   **Logging Config**: `DEFAULT_LOGGING_CONTEXT` including flags like `enabled`, `log_directory`.
-   **`EXCLUDED_FILES: Set[str]`**: A set of filenames to ignore during operations like adding a directory to context.
-   **`EXCLUDED_EXTENSIONS: Set[str]`**: A set of file extensions to ignore.

### System Prompt
-   **`load_system_prompt() -> str`**:
    -   **Description:** Loads the main system prompt for the AI from `system_prompt.txt`.
    -   **Returns:** The content of the system prompt file.
-   **`DEFAULT_SYSTEM_PROMPT: str`**: A fallback system prompt string defined directly in the code, including dynamic OS information.
-   **`SYSTEM_PROMPT: str`**: The actual system prompt used by the application, loaded by `load_system_prompt()`.

### Fuzzy Matching Availability
-   **`FUZZY_AVAILABLE: bool`**: A boolean indicating whether the `thefuzz` library was successfully imported and is available for use.

### Function Calling Tools Definition
-   **`tools: List[Dict[str, Any]]`**: A list of dictionaries defining the tools available to the Gemini model. Each tool has a name, description, and parameter schema, following a format similar to OpenAI's function calling. Examples include `read_file`, `create_file`, `git_commit`, `run_bash`.
-   **`convert_tools_to_gemini() -> List[google.genai.types.Tool]`**:
    -   **Description:** Converts the `tools` list from the internal/OpenAI-like format to the `google.genai.types.Tool` format required by the Gemini API.
    -   **Returns:** A list of `types.Tool` objects.

### Initialization
-   The script initializes `model_context`, `security_context`, and `logging_context` with values derived from the loaded `config` or defaults.

## Important Variables/Constants

-   **`config: Dict[str, Any]`**: The primary dictionary holding all settings loaded from `config.json`. Many other constants are derived from this.
-   **`os_info: Dict[str, Any]`**: Contains runtime information about the operating system.
-   **`SYSTEM_PROMPT: str`**: The guiding prompt for the AI, crucial for defining its behavior and capabilities.
-   **`tools: List[Dict[str, Any]]`**: Defines the set of functions the AI can request to call.
-   **`DEFAULT_MODEL: str`**: The identifier for the default Gemini model to be used.
-   **`REASONER_MODEL: str`**: The identifier for the reasoner/more advanced Gemini model.
-   **`EXCLUDED_FILES` and `EXCLUDED_EXTENSIONS`**: Sets defining what files and types should be ignored by certain operations.

## Usage Examples

This file is not meant to be run directly. It is imported by other modules, primarily `main.py`, to access configuration values and context.

```python
# In main.py or utils.py
from config import SYSTEM_PROMPT, tools, base_dir, git_context, MAX_FILE_CONTENT_SIZE_CREATE

# Use the imported configurations
print(f"Current base directory: {base_dir}")
if git_context['enabled']:
    print("Git is enabled.")

# Gemini API call might use SYSTEM_PROMPT and tools
# client.generate_content(..., system_instruction=SYSTEM_PROMPT, tools=gemini_tools_list)
```

## Dependencies and Interactions

-   **Internal Dependencies:** None directly, but it's a central piece for other modules.
-   **External Libraries:**
    -   `json`: For parsing `config.json`.
    -   `platform`: For gathering OS information.
    -   `pathlib`: For path manipulations.
    -   `google.generativeai.types` (conditionally, within `convert_tools_to_gemini`): For creating Gemini-specific tool objects.
    -   `thefuzz` (optional): For fuzzy string matching capabilities. Its absence sets `FUZZY_AVAILABLE` to `False`.
-   **Interactions:**
    -   Reads `config.json` to load user-defined settings.
    -   Reads `system_prompt.txt` to load the AI's guiding instructions.
    -   Provides configuration data and initial state (like `os_info`, `tools` definitions, `SYSTEM_PROMPT`) to `main.py` and `utils.py`.
    -   The `tools` definition here is critical for how `main.py` allows the AI to interact with the system (file operations, Git, etc.).
```
