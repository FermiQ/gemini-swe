# main.py

## Overview

This file is the main entry point and core logic for the Gemini Engineer application. It handles user commands, manages the conversation flow with the Gemini AI, executes tool calls (like file operations, Git commands, and shell commands), and interacts with various modules for configuration and utility functions.

## Key Components

### Pydantic Models
-   **`FileToCreate(BaseModel)`**
    -   **Description:** Defines the structure for a file to be created, including its path and content.
    -   **Attributes:**
        -   `path: str`: The path where the file should be created.
        -   `content: str`: The content to write into the file.
-   **`FileToEdit(BaseModel)`**
    -   **Description:** Defines the structure for editing a file, including the path, the original snippet to find, and the new snippet to replace it with.
    -   **Attributes:**
        -   `path: str`: The path of the file to be edited.
        -   `original_snippet: str`: The snippet of text to search for in the file.
        -   `new_snippet: str`: The snippet of text to replace the original snippet with.

### File Operations
-   **`create_file(path: str, content: str, require_confirmation: bool = True) -> None`**
    -   **Description:** Creates or overwrites a file with the given content. It includes checks for file size limits, path validity, and prompts for confirmation when overwriting existing files.
    -   **Parameters:**
        -   `path: str`: File path.
        -   `content: str`: File content.
        -   `require_confirmation: bool`: If True, prompt for confirmation before overwriting.
-   **`add_directory_to_conversation(directory_path: str, conversation_history: List[Dict[str, Any]]) -> None`**
    -   **Description:** Scans a directory and adds its file contents to the conversation history, respecting exclusion rules and size limits.
    -   **Parameters:**
        -   `directory_path: str`: Path to the directory to scan.
        -   `conversation_history: List[Dict[str, Any]]`: The conversation history list to append file contexts to.

### Git Operations
-   **`stage_file(file_path_str: str) -> bool`**
    -   **Description:** Stages a file for a Git commit if Git is enabled and not skipped.
    -   **Parameters:** `file_path_str: str`: Path to the file to stage.
    -   **Returns:** `bool`: True if staging was successful, False otherwise.
-   **`get_git_status_porcelain() -> Tuple[bool, List[Tuple[str, str]]]`**
    -   **Description:** Retrieves the Git status in porcelain format.
    -   **Returns:** `Tuple[bool, List[Tuple[str, str]]]`: A tuple containing a boolean indicating if there are changes, and a list of tuples with (status_code, filename).
-   **`create_gitignore() -> None`**
    -   **Description:** Creates a comprehensive `.gitignore` file if one doesn't exist, with options for custom patterns.
-   **`user_commit_changes(message: str) -> bool`**
    -   **Description:** Commits staged changes with a given message. Prompts the user if no changes are staged or if there are unstaged changes.
    -   **Parameters:** `message: str`: The commit message.
    -   **Returns:** `bool`: True if the commit was successful or an action (like aborting) was taken.

### Command Handlers
-   A suite of functions prefixed with `try_handle_` (e.g., `try_handle_git_command`, `try_handle_add_command`) or ending with `_cmd` (e.g., `show_git_status_cmd`, `initialize_git_repo_cmd`).
    -   **Description:** These functions parse user input for specific slash commands (e.g., `/git status`, `/add <file>`, `/help`) and execute the corresponding actions, such as interacting with Git, managing files, clearing context, or displaying help information.

### Gemini API Helper Functions
-   **`convert_conversation_to_gemini(...) -> List[types.Content]`**
    -   **Description:** Converts the internal conversation history format to the format required by the Gemini API.
-   **`call_gemini_api(...) -> Tuple[str, List[Dict[str, Any]], Any]`**
    -   **Description:** Makes a call to the Gemini API with the current conversation history, system prompt, and tools. It handles the request and extracts the response content and any tool calls made by the AI.
    -   **Returns:** A tuple containing the response text, a list of tool calls, and the raw API response object.

### LLM Tool Handler Functions
-   A suite of functions prefixed with `llm_` (e.g., `llm_git_init`, `llm_read_file`).
    -   **Description:** These functions are wrappers around application functionalities (like Git operations or file system interactions) that are exposed as tools to the Gemini model. They are called when the AI decides to use a specific tool.
-   **`execute_function_call_dict(tool_call_dict: Dict[str, Any]) -> str`**
    -   **Description:** Takes a tool call dictionary (as provided by the AI) and executes the corresponding `llm_` function. It includes security checks for potentially sensitive operations like running shell commands.
    -   **Parameters:** `tool_call_dict: Dict[str, Any]`: The tool call information.
    -   **Returns:** `str`: The result of the tool execution, to be sent back to the AI.

### Main Loop & Entry Point
-   **`initialize_application() -> None`**
    -   **Description:** Initializes the application state, including detecting available shells and checking for an existing Git repository.
-   **`main_loop() -> None`**
    -   **Description:** The main interactive loop of the application. It prompts the user for input, handles commands, sends requests to the Gemini API, processes responses (including executing tool calls), and manages the conversation history.
-   **`main() -> None`**
    -   **Description:** The primary entry point of the script. It prints a welcome message, initializes the application, and starts the main loop.

## Important Variables/Constants

-   **`client = genai.Client(...)`**: The initialized Google Gemini API client.
-   **`prompt_session = PromptSession(...)`**: The `prompt_toolkit` session for handling user input with rich features.
-   **`conversation_history: List[Dict[str, Any]]`**: Stores the ongoing conversation between the user, the AI, and tool responses. This is the primary context sent to the Gemini API.
-   **`base_dir: Path`**: The current base directory for file operations. Can be changed by the user via the `/folder` command. Initialized from `config.base_dir`.
-   **`SYSTEM_PROMPT: str`**: The initial system prompt that guides the AI's behavior. Imported from `config`.
-   **`tools: List[Dict[str, Any]]`**: A list of tool definitions available to the AI. Imported from `config`.
-   Various constants related to command prefixes (e.g., `ADD_COMMAND_PREFIX`, `COMMIT_COMMAND_PREFIX`), file size limits (e.g., `MAX_FILE_CONTENT_SIZE_CREATE`), and excluded files/extensions, all imported from `config`.

## Usage Examples

The application is run directly from the command line:
```bash
python main.py
```
Once running, the user interacts with it through a prompt:
```
💬 You: Create a new Python file named 'hello.py' that prints 'Hello, World!'.
```
The AI might then respond or use a tool:
```
🤖 Gemini: Okay, I will create that file for you.
... (Tool call to create_file executed) ...
[bold blue]✓[/bold blue] Created file at '[bright_cyan]hello.py[/bright_cyan]'
```
Users can also use slash commands:
```
💬 You: /git status
💬 You: /add my_script.py
💬 You: /help
```

## Dependencies and Interactions

-   **Internal Dependencies:**
    -   `config.py`: For loading configuration settings, API keys, model names, tool definitions, context variables (like `os_info`, `git_context`), and constants.
    -   `utils.py`: For various utility functions like console output, path normalization, file reading, fuzzy matching, shell command execution, context management, and logging.
-   **External Libraries:**
    -   `google-generativeai`: For interacting with the Gemini API.
    -   `pydantic`: For data validation and settings management through models (`FileToCreate`, `FileToEdit`).
    -   `python-dotenv`: For loading environment variables (like API keys).
    -   `rich`: For rich text formatting in the console.
    -   `prompt_toolkit`: For creating an interactive command-line interface.
-   **Interactions:**
    -   Interacts heavily with the file system for reading, creating, and editing files based on AI instructions or user commands.
    -   Interacts with Git through `subprocess` calls to perform version control operations.
    -   Interacts with the operating system shell (Bash, PowerShell) via `subprocess` to execute arbitrary commands as instructed by the AI (with user confirmation).
    -   The core interaction is with the Gemini API, sending conversation history and receiving text responses or tool usage requests.
```
