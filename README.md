# sc-agent-manager

`sc-agent-manager` is a repository for manual orchestration of three LLMs working on the same project:
- `CLAUDE`
- `CODEX`
- `GEMINI`

The human operator is the orchestrator. Models do not self-orchestrate.
All macro behavior is defined in `macros.md`.

## Template Repository Policy
This repository is intentionally kept as a clean template:

- no personal task history
- no session logs or solution artifacts in version control
- no user-specific local settings

Use it as a drop-in orchestrator layer for any project, then run sessions locally without committing runtime traces.

## How It Works in Practice
The idea behind this repository is simple: collect perspectives and proposals from multiple language models around one shared task context defined in `prompt.md`.

You describe a goal, problem, request, or desired outcome in `prompt.md` in any format you prefer. Connected CLIs (in this setup: Codex, Gemini, and Claude) can run in either Observer or Executor mode via macro commands.

When you are ready to start analysis or gather feedback, run `tellmeobs` in each active terminal session. In Observer mode, an agent does not modify project files; it investigates and produces a `solutions` document with proposed changes and prompt-specific recommendations.

You can run multiple observers in parallel. In practice, three observers (one per model) are usually enough. By running `upd` between observer sessions, you synchronize status and ACK evidence across artifacts. If you want observers to review each other's proposals and refine them, run another `tellmeobs` round.

When you are ready to implement, run `tellmeexe`. Any observer can switch to Executor mode. The executor prepares an implementation plan and pauses. You can review the plan or skip it, then instruct the executor to proceed.

After implementation, observers can run `upd` to synchronize implementation status. If you want improvement suggestions, run a new `tellmeobs` cycle. The executor can run `upd` to refresh state and then use `tellmeexe` to apply approved corrections.

An executor session ends with a detailed ACK report and a commit proposal. All steps are logged, so new sessions can quickly understand what happened in the repository.

Best practice: avoid manual edits while a coordinated multi-agent cycle is running. If you edit files manually, notify all active agents.

This system is not intended to be autonomous. It is designed to keep human control while leveraging different model strengths and token economics. I use this orchestration model for writing, libraries, scripts, and review tasks. It works well in practice and keeps the process transparent, even in areas where I am not deeply technical.

## What This Project Solves

This repository is designed to solve practical coordination tasks in multi-agent work:

1. Deterministic boot and role startup
2. Clear separation between analysis (`OBS`) and implementation (`EXE`)
3. Shared task intake through `prompt.md`
4. Standardized proposal lifecycle (`PROPOSED` -> `IMPLEMENTED`)
5. Traceable implementation evidence through ACK files
6. Safe, user-approved commit/push flow
7. Session reset and hygiene across logs, contexts, and solutions

## Core Project Tasks

The project is focused on these operational tasks:

1. Task ingestion and synchronization
- Keep one active task source in `prompt.md`
- Re-sync agents after prompt updates (`prompt`, `upd`)

2. Multi-observer analysis
- Run one or more `*-OBS` agents
- Generate structured proposals in `AGENTS/<MODEL>-OBS/solutions/`

3. Proposal review and decision
- Human reviews proposals
- Human selects what moves to implementation

4. Controlled execution
- Run `*-EXE` to gather selected proposals
- Build execution plan (`master_plan.md`)
- Implement only after user confirmation

5. Evidence and audit trail
- Write implementation ACK files in `AGENTS/<MODEL>-EXE/solutions/ack/`
- Keep append-only logs for traceability

6. Session maintenance
- Synchronize registry and status with `upd`
- Reset session state with `cleanup` when needed

## Operating Model

Two runtime roles are used:

- `Observer (OBS)`: scans the project and prepares proposals
- `Executor (EXE)`: consolidates proposals, creates plan, implements approved changes

The orchestrator (human) decides:
- which model runs OBS/EXE
- which proposals are accepted
- whether commit/push should happen

## Supported Agent Layout

Default model set for this repository:
- `CLAUDE`
- `CODEX`
- `GEMINI`

Each model can have:
- `AGENTS/<MODEL>-OBS/`
- `AGENTS/<MODEL>-EXE/`
- `AGENTS/<MODEL>_CONTEXT.md`

## Main Commands

These are macro commands (not skills), implemented in `macros.md`:

| Command | Purpose | Typical Use |
|---|---|---|
| `prompt` | Re-read `prompt.md`, sync context, route role if needed, then execute role logic | After editing `prompt.md` |
| `tellmeobs` | Start observer flow and produce proposal(s) | Analysis phase |
| `tellmeexe` | Start executor flow, build plan, implement approved work | Execution phase |
| `upd` | Synchronize state (`git status`, registry, proposal/ACK sync) | Start of session, before/after major steps |
| `cleanup` | Reset session artifacts (logs/solutions/contexts/prompt stub) | End of iteration |

## Recommended Workflow

1. Write or update `prompt.md` with the current task.
2. Run `upd` (or `prompt`) to synchronize state.
3. Run `tellmeobs` in one or more models.
4. Review proposals in `AGENTS/*-OBS/solutions/`.
5. Run `tellmeexe` in the chosen executor model.
6. Validate implementation results.
7. Optionally run `cleanup` to reset session state.

## Proposal Lifecycle

Typical states and artifacts:

- Proposal file created with `STATUS: PROPOSED`
- Executor implements and creates ACK:
  - `AGENTS/<MODEL>-EXE/solutions/ack/<proposal_id>.ack`
- Observer sync (`upd`) can mark proposal as implemented with evidence

This gives end-to-end traceability from idea -> implementation evidence.

## Repository Structure

```text
.
|-- AGENTS/
|   |-- AGENTS.md
|   |-- <MODEL>_CONTEXT.md
|   |-- <MODEL>-OBS/
|   |   |-- instructions/
|   |   |-- logs/
|   |   `-- solutions/
|   `-- <MODEL>-EXE/
|       |-- logs/
|       `-- solutions/
|-- .claude/
|-- .codex/
|-- .gemini/
|-- macros.md
|-- prompt.md
|-- HOW_TO_USE.md
`-- README.md
```

## Safety and Governance

Key safety rules enforced by macros:

- Boot order is strict: `LOCAL CONFIG -> MACROS -> PROMPT`
- Macro definitions must be loaded from disk (no cached macro behavior)
- Search-before-action policy for incoming requests
- No autonomous commit/push by default (user approval required)
- Cross-agent write isolation (except explicitly allowed shared files)

## Quick Start

1. Open `prompt.md` and write your task.
2. In your agent session, run:
- `upd`
- `tellmeobs`
3. Review generated proposal(s).
4. Run `tellmeexe` to implement approved proposal(s).

## License

This project is licensed under the MIT License.
See [LICENSE](LICENSE).

## Author

Daniel Kein  
https://www.linkedin.com/in/daniel-kein/
