# HOW_TO_USE

This repository is designed for manual orchestration of three language models while working on project tasks:
- `CLAUDE`
- `CODEX`
- `GEMINI`

The user acts as the orchestrator: defines tasks, chooses agent roles, reviews proposals, and confirms implementation.

Source of macro rules and behavior: `macros.md`.

## Template Baseline
This repository is meant to stay clean in version control:

- keep `prompt.md` in a neutral "No active task" state
- keep `AGENTS/*/logs/` and `AGENTS/*/solutions/` empty except `.gitkeep`
- avoid committing machine-specific local settings

## Where Everything Lives

- `prompt.md` - current task from the user.
- `AGENTS/<MODEL>-OBS/solutions/` - observer proposals (analysis/proposal).
- `AGENTS/<MODEL>-EXE/solutions/` - implementation plans and results.
- `AGENTS/<MODEL>-EXE/solutions/ack/` - ACK files confirming implementation.
- `AGENTS/AGENTS.md` - agent activity registry.

## Commands (Purpose + Example)

| Command | Purpose | Example |
|---|---|---|
| `prompt` | Sync agent context with the latest `prompt.md`; select role (OBS/EXE) if needed. | After editing `prompt.md`, send: `prompt` |
| `tellmeobs` | Start OBS mode: scan the repository and produce proposals in `AGENTS/<MODEL>-OBS/solutions/`. | `tellmeobs` |
| `tellmeexe` | Start EXE mode: collect `STATUS: PROPOSED`, build `master_plan`, and implement after approval. | `tellmeexe` |
| `upd` | Synchronize state: `git status`/diff, LastActive, ACK checks, proposal status sync. | `upd` |
| `cleanup` | Prepare a clean session: clear logs/solutions, reset context files, reset `prompt.md`. | `cleanup` |

## Recommended Workflow

1. Write the task in `prompt.md`.
2. Run `upd` (or `prompt`) to synchronize state.
3. Run `tellmeobs` in one or more models.
4. Review proposals in `AGENTS/*-OBS/solutions/`.
5. Choose an executor and run `tellmeexe`.
6. Validate the implemented changes.
7. Optionally run `cleanup` at the end of the iteration.

## Mini Scenario

1. You update `prompt.md` with a new task.
2. You send `prompt` to `CODEX`.
3. You run `tellmeobs` in `CLAUDE` and `GEMINI`.
4. You select the best proposal.
5. You run `tellmeexe` in `CODEX` for implementation.
6. You verify the result and decide whether `cleanup` is needed.
