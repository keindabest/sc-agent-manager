# Gemini CLI Tools Reference

This document provides a brief overview of the tools and sub-agents available to the Gemini CLI agent.

## Core File Operations
- `read_file`: Reads the content of a file (text, images, PDF, etc.).
- `write_file`: Creates or overwrites a file with new content.
- `replace`: Performs precise text replacement within a file using context matching.
- `list_directory`: Lists files and folders in a specified path.
- `glob`: Finds files matching specific patterns (e.g., `**/*.md`).

## System & Search
- `run_shell_command`: Executes bash commands. Used for builds, tests, and git operations.
- `grep_search`: Searches for text patterns across multiple files (regex supported).
- `google_web_search`: Performs a Google search to find information online.
- `web_fetch`: Fetches and processes content from URLs.

## Specialized Sub-Agents
- `codebase_investigator`: Deeply analyzes the codebase for architecture mapping, bug root-cause analysis, and system-wide understanding.
- `cli_help`: Answers questions about Gemini CLI features, documentation, and configuration.

## Interaction & Memory
- `ask_user`: Prompts the user with questions (multi-choice, text, or yes/no).
- `save_memory`: Saves specific user preferences or facts to long-term memory for future sessions.
- `activate_skill`: Activates specialized expert skills (e.g., `skill-creator`).

---
*Generated on 2026-02-12*
