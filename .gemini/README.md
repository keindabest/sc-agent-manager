# Gemini Project Settings

This directory contains workspace-specific configurations for the Gemini CLI.

### Behavioral Mandates
The core instructions and custom macros for this project are defined in the root **`GEMINI.md`** file. 
The AI agent reads this file automatically at the start of every session to align with project-specific workflows (e.g., handling `tellmeobs`, `tellmeexe`, etc.).

### Global Preferences vs. Workspace Settings
1. **Workspace Mandates (`GEMINI.md`):** Used for project-specific rules, coding styles, and workflows. These apply only to this repository.
2. **Global Memories (`save_memory`):** Use this tool to save facts about yourself or your preferences that should persist across **ALL** workspaces and projects. 
   - *Example command:* "Save memory: I always use Cursor for editing files."
   - These memories are stored globally in your user profile and are retrieved in every session, regardless of the folder.

### How to View or Edit Mandates
- **In Chat:** Use `read_file GEMINI.md`.
- **In Editor:** To open the mandates file directly in **Cursor**, run the following command in your terminal:
  ```bash
  cursor GEMINI.md
  ```

### Local Configuration
- `settings.json`: Defines technical preferences like `approvalMode`. Currently set to `yolo` for this workspace to allow autonomous execution.
