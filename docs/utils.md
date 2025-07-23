# utils.py

## Overview

The `utils.py` file provides a collection of helper functions used throughout the Gemini Engineer application. These utilities cover a range of functionalities including system interaction, context management for the AI, file operations, string manipulation (fuzzy matching), shell command execution, and logging.

## Key Components

### Console and System Utilities
-   **`console = Console()`**: An instance of `rich.console.Console` for enhanced terminal output.
-   **`detect_available_shells() -> None`**:
    -   **Description:** Checks for the presence of common shells (bash, powershell, zsh, cmd) on the system and updates `os_info['shell_available']` in `config.py`.
-   **`get_prompt_indicator(...) -> str`**:
    -   **Description:** Generates a string for the user prompt, indicating current model, Git branch (if active), and context usage status (normal, warning, critical).

### Context Management Utilities
-   **`estimate_token_usage(...) -> Tuple[int, Dict[str, int]]`**:
    -   **Description:** Estimates the number of tokens in the conversation history based on character count and message structure.
    -   **Returns:** A tuple with the total estimated tokens and a breakdown by role (system, user, assistant, tool).
-   **`get_context_usage_info(...) -> Dict[str, Any]`**:
    -   **Description:** Provides a comprehensive dictionary of context usage statistics, including token counts, percentages, and health status relative to model limits.
-   **`smart_truncate_history(...) -> List[Dict[str, Any]]`**:
    -   **Description:** Intelligently truncates the conversation history to stay within token limits, prioritizing recent messages, system prompts, and complete tool call sequences.
-   **`validate_tool_calls(...) -> List[Dict[str, Any]]`**:
    -   **Description:** Checks a list of AI-generated tool calls for validity (e.g., presence of ID, function name, valid JSON arguments).
-   **`add_file_context_smartly(...) -> bool`**:
    -   **Description:** Adds file content to the conversation history as a system message. Manages the number of file contexts, avoids duplicates, and handles large files or pending tool calls gracefully.

### File and Path Utilities
-   **`normalize_path(path_str: str, allow_outside_project: bool = False) -> str`**:
    -   **Description:** Resolves and normalizes a file path string relative to `base_dir`. Includes security checks to prevent path traversal outside the project directory unless explicitly allowed.
    -   **Returns:** A normalized absolute path string.
-   **`is_binary_file(file_path: str, peek_size: int = 1024) -> bool`**:
    -   **Description:** Checks if a file is likely a binary file by looking for null bytes in the first `peek_size` bytes.
-   **`read_local_file(file_path: str) -> str`**:
    -   **Description:** Reads the content of a local file (UTF-8 encoded).
-   **`get_directory_tree_summary(...) -> str`**:
    -   **Description:** Generates a string representation of a directory tree up to a specified depth and entry count.

### Fuzzy Matching Utilities (Conditional on `thefuzz` library)
-   **`find_best_matching_file(...) -> Optional[str]`**:
    -   **Description:** If `FUZZY_AVAILABLE` is True, attempts to find the best file match for a potentially misspelled or partial user-provided path within a root directory.
-   **`apply_fuzzy_diff_edit(...) -> None`**:
    -   **Description:** Applies an edit to a file by replacing `original_snippet` with `new_snippet`. First tries an exact match; if that fails and `FUZZY_AVAILABLE` is True, it uses fuzzy matching to find the closest snippet to replace.

### Shell Command Utilities
-   **`run_bash_command(command: str) -> Tuple[str, str, int]`**:
    -   **Description:** Executes a bash command. Checks for bash availability (including WSL or Git Bash on Windows).
    -   **Returns:** A tuple `(stdout, stderr, returncode)`.
-   **`run_powershell_command(command: str) -> Tuple[str, str, int]`**:
    -   **Description:** Executes a PowerShell command. Checks for PowerShell availability (Windows PowerShell or PowerShell Core).
    -   **Returns:** A tuple `(stdout, stderr, returncode)`.

### Logging Utilities
-   **`initialize_logging() -> None`**: Sets up the logging directory and initial log file for the current session if logging is enabled.
-   **`log_user_message(content: str, model: str) -> None`**: Logs a user's message.
-   **`log_api_response(response, ..., tool_calls: List[Dict[str, Any]] = None) -> None`**: Logs the AI's response, including token usage and any tool calls made.
-   **`log_tool_execution(tool_call: Dict[str, Any], result: str) -> None`**: Logs the execution of a tool call and its result.
-   **`toggle_logging() -> str`**: Enables or disables logging and returns a status message.
-   **`get_logging_status() -> str`**: Returns a string indicating the current logging status and log file name if active.
-   **`_write_log_entry(entry: Dict[str, Any]) -> None`**: Internal function to write a JSON log entry to the current log file.
-   **`_rotate_log_file() -> None`**: Internal function to start a new log file when the current one exceeds size limits.
-   **`_cleanup_old_logs() -> None`**: Internal function to delete older log files to stay within `max_log_files` limit.

## Important Variables/Constants

-   **`console: Console`**: The Rich console instance used for all styled terminal output.
-   Constants imported from `config.py` are heavily used, such as `base_dir`, `FUZZY_AVAILABLE`, `MIN_FUZZY_SCORE`, `MAX_HISTORY_MESSAGES`, `logging_context`, etc. These govern the behavior of many utility functions.

## Usage Examples

These functions are typically called by `main.py` or other parts of the application.

```python
# In main.py, when a user issues an /add command:
from utils import normalize_path, read_local_file, add_file_context_smartly, find_best_matching_file

user_path = "src/my_modul.py" # User might misspell
corrected_path = find_best_matching_file(base_dir, user_path)
if corrected_path:
    try:
        content = read_local_file(corrected_path)
        add_file_context_smartly(conversation_history, corrected_path, content)
    except Exception as e:
        console.print(f"Error: {e}")

# In main.py, before calling the AI:
from utils import get_prompt_indicator, smart_truncate_history
conversation_history = smart_truncate_history(conversation_history, model_name=current_model)
prompt_text = get_prompt_indicator(conversation_history, current_model) + " You: "

# When the AI requests to run a bash command:
from utils import run_bash_command
stdout, stderr, retcode = run_bash_command("ls -la")
if retcode == 0:
    tool_result = stdout
else:
    tool_result = f"Error: {stderr}"
```

## Dependencies and Interactions

-   **Internal Dependencies:**
    -   `config.py`: For accessing a wide range of configuration values (e.g., `os_info`, `base_dir`, `model_context`, `logging_context`, various limits and thresholds, `FUZZY_AVAILABLE`).
-   **External Libraries:**
    -   `os`, `sys`, `json`, `re`, `time`, `subprocess`, `platform`, `pathlib`: Standard Python libraries for system interaction, file operations, and data handling.
    -   `rich`: For all console output formatting.
    -   `thefuzz` (optional, via `config.FUZZY_AVAILABLE`): For fuzzy string matching in `find_best_matching_file` and `apply_fuzzy_diff_edit`.
-   **Interactions:**
    -   Interacts with the file system for reading files (`read_local_file`), checking file types (`is_binary_file`), normalizing paths (`normalize_path`), and managing log files.
    -   Interacts with the operating system shell via `subprocess` to run bash and PowerShell commands.
    -   Manipulates and analyzes the `conversation_history` list for context management and truncation.
    -   Provides feedback to the user via the `console` object.
```
