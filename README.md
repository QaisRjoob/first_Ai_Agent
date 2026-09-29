<div align="center">

# 🤖 First AI Agent

**A command-line AI coding agent that can explore, read, write, and run Python code on its own, using LLM tool calling.**

![Python](https://img.shields.io/badge/Python-3.14-3776AB?style=for-the-badge&logo=python&logoColor=white)
![OpenAI SDK](https://img.shields.io/badge/OpenAI_SDK-2.44-412991?style=for-the-badge&logo=openai&logoColor=white)
![OpenRouter](https://img.shields.io/badge/OpenRouter-LLM_Gateway-6566F1?style=for-the-badge)
![uv](https://img.shields.io/badge/uv-package_manager-DE5FE9?style=for-the-badge)

</div>

---

## 📖 Overview

**First AI Agent** is a lightweight, from-scratch implementation of an **agentic loop**. You give it a task in plain English, for example *"fix the bug in the calculator"*, and the agent decides which tools to call, looks at the results, and keeps going until it has a final answer.

It is built with the **OpenAI Python SDK** pointed at **[OpenRouter](https://openrouter.ai/)**, so it can run on any tool-calling model OpenRouter supports. It uses the free router (`openrouter/free`) by default.

The agent works inside a **sandboxed working directory** (`./calculator`), a small sample project it can inspect, change, and test.

## ✨ Features

- 🔁 **Agentic loop:** runs up to 20 reason → act → observe cycles per request
- 🛠️ **Tool calling:** four tools declared with JSON Schema
- 🔒 **Sandboxed file access:** every path is resolved and checked against the working directory, so the model can't reach outside it (`../` path-traversal is blocked)
- ⏱️ **Safe execution:** Python scripts run in a subprocess with a 30-second timeout, and both stdout and stderr are captured
- 📏 **Output limits:** file reads are truncated at 10,000 characters to protect the context window
- 🔍 **Verbose mode:** shows token usage, each function call with its arguments, and each tool result

## 🧰 Available Tools

| Tool | Description |
|------|-------------|
| `get_files_info` | Lists files in a directory with their size and whether each one is a directory |
| `get_file_content` | Reads a file's contents (max 10,000 chars) |
| `run_python_file` | Runs a `.py` file with optional CLI arguments |
| `write_file` | Creates or overwrites a file, creating parent directories if needed |

## 🏗️ How It Works

```text
            ┌──────────────────────┐
  prompt ─▶ │   main.py            │
            │   (agent loop ×20)   │
            └─────────┬────────────┘
                      │ messages + tool schemas
                      ▼
            ┌──────────────────────┐
            │   LLM via OpenRouter │
            └─────────┬────────────┘
          tool_calls? │
         ┌────yes─────┴─────no─────┐
         ▼                         ▼
┌──────────────────┐        ┌───────────────┐
│ call_function.py │        │ Final response│
│ (dispatcher)     │        └───────────────┘
└────────┬─────────┘
         ▼
┌──────────────────────────────┐
│ functions/*  (sandboxed to   │
│ ./calculator)                │
└────────┬─────────────────────┘
         │ tool result appended to messages
         └──────────────▶ back to the LLM
```

1. The **system prompt** (`prompts.py`) tells the model what it can do and when to use each tool.
2. The model either answers directly or asks for one or more **tool calls**.
3. `call_function.py` injects the sandboxed `working_directory`, runs the matching function, and returns the result as a `tool` message.
4. The loop keeps going until the model gives a final answer, or it stops after 20 iterations.

## 📂 Project Structure

```text
first_Ai_Agent/
├── main.py                  # CLI entry point & agent loop
├── prompts.py               # System prompt
├── call_function.py         # Tool registry & dispatcher
├── functions/
│   ├── get_files_info.py    # List directory contents
│   ├── get_file_content.py  # Read file contents
│   ├── run_python_file.py   # Execute Python scripts
│   └── write_file.py        # Write / overwrite files
├── calculator/              # Sandboxed sample project the agent works on
│   ├── main.py              # Calculator CLI (JSON output)
│   ├── tests.py             # unittest suite
│   └── pkg/
│       ├── calculator.py    # Infix expression evaluator with operator precedence
│       └── render.py        # JSON output formatter
├── test_*.py                # Manual tests for each tool
├── pyproject.toml
└── uv.lock
```

## 🚀 Getting Started

### Prerequisites

- **Python 3.14+**
- **[uv](https://docs.astral.sh/uv/)** (recommended) or pip
- An **[OpenRouter API key](https://openrouter.ai/keys)**

### Installation

```bash
git clone https://github.com/QaisRjoob/first_Ai_Agent.git
```

```bash
cd first_Ai_Agent
```

```bash
uv sync
```

### Configuration

Create a `.env` file in the project root (it's already listed in `.gitignore`):

```env
OPENROUTER_API_KEY=your_api_key_here
```

## 💻 Usage

```bash
uv run main.py "<your prompt>" [--verbose]
```

### Examples

```bash
uv run main.py "what files are in the root?"
```

```bash
uv run main.py "read the contents of pkg/calculator.py"
```

```bash
uv run main.py "run the calculator tests"
```

```bash
uv run main.py "how does the calculator render results to the console?" --verbose
```

Example output:

```text
 - Calling function: get_files_info
 - Calling function: get_file_content
Final response:
The calculator formats results as JSON using format_json_output() in pkg/render.py ...
```

## 🧪 Testing

Run the manual tests for each tool:

```bash
uv run test_get_files_info.py
```

```bash
uv run test_get_file_content.py
```

```bash
uv run test_write_file.py
```

```bash
uv run test_run_python_file.py
```

Run the calculator's unit tests:

```bash
cd calculator && uv run tests.py
```

## 🔐 Security Notes

- The model **never chooses** the working directory. The dispatcher injects it on every call.
- Every path is normalized and checked with `os.path.commonpath`, which blocks traversal outside the sandbox.
- Only `.py` files can be run, and each run times out after 30 seconds.
- ⚠️ This is a learning project. The agent can overwrite files inside its sandbox, so don't point it at code you care about without version control.

## 🗺️ Roadmap

- [ ] Make the model and working directory configurable from the CLI
- [ ] Add a confirmation step before writes and executions
- [ ] Convert the manual tool tests into a `pytest` suite
- [ ] Add streaming output

## 👤 Author

**Qais Rjoob**

[![GitHub](https://img.shields.io/badge/GitHub-QaisRjoob-181717?style=flat&logo=github)](https://github.com/QaisRjoob)

---

<div align="center">

⭐ If you found this project useful, consider giving it a star!

</div>
