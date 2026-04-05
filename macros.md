# AGENTS ORCHESTRATOR MACROS
Version: 1.16
Mode: Deterministic
All rules below are STRICT.

INTRO (self-contained)
- For `tellmeobs`, `tellmeexe`, `prompt`, `upd`, `cleanup`, boot order is STRICT:
  LOCAL CONFIG -> MACROS -> PROMPT.
- Read local config by `<NAME>` first (see Global Rule #10), then read and execute root `macros.md`.
- Built-in/cached macro definitions for these commands are forbidden; always reload from disk.
- If `macros.md` is missing/unreadable: STOP with an error.
- At each macro run, log `Version:` and `Boot: ConfigFirst <CONFIG_DIR|NONE>`.

=====================================================================
GLOBAL RULES
=====================================================================

1. All agents operate inside /AGENTS/.
2. Allowed identifiers for <NAME>: **CLAUDE, CODEX, KIMI, GEMINI** only.
   - At the start of `tellmeobs`/`tellmeexe`, auto-detect which of these four models you actually are (read the system/developer instructions or other runtime metadata that states the current model).
   - If the detected name is inside the allowed list, set `<NAME>` to that value for the entire session (folders, logs, IDs, context).
   - If you cannot determine the model or the result is outside this list, **STOP immediately** and tell the user: `"ERROR: Unknown agent identity (<detected_value>). Allowed: CLAUDE/CODEX/KIMI/GEMINI. Update macros or clarify before continuing."`
3. Agents may only write inside their own subfolder:
   /AGENTS/<NAME>-OBS/
   /AGENTS/<NAME>-EXE/
4. Never modify another agent's folder or files.
   Exception: Updating /AGENTS/AGENTS.md, /AGENTS/<NAME>_CONTEXT.md.
5. Always exclude from scans:
   .git/, node_modules/, dist/, build/, .next/, out/, vendor/, tmp/, coverage/
6. All timestamps: YYYY-MM-DD HH:mm +0100
7. All IDs format:
   <NAME>-OBS-YYYYMMDD-NNN
   <NAME>-EXE-YYYYMMDD-NNN
8. All logs must be append-only.
9. **NO AUTONOMOUS PUSH/COMMIT:**
   - По умолчанию: перед ЛЮБЫМ `git commit` и/или `git push` в рамках TPCP (Savepoint/Final) требуется явное согласие пользователя в чате.
   - Без согласия: не выполнять ни commit, ни push — работа остаётся локально (dirty или staged, но не зафиксирована удалённо).
   - Исключения (только если явно задано в `prompt.md`):
     * `PUSH_MODE: AUTO` — разрешить TPCP commit+push без дополнительных вопросов.
     * `PUSH_MODE: FINAL_MANUAL` — при финализации выполнить commit без push (всё равно требуется согласие пользователя на commit).
10. **BOOT ORDER & LOCAL CONFIG (ALL AGENTS):**
    - BOOT ORDER (strict): LOCAL CONFIG → MACROS → PROMPT.
    - Local config directories by agent name (if present):
      * `<NAME>` = CODEX → `.codex/`
      * `<NAME>` = GEMINI → `.gemini/`
      * `<NAME>` = CLAUDE → `.claude/` (optional)
      * `<NAME>` = KIMI → `.kimi/` (optional)
    - Read order inside config dir (if it exists): `AGENTS.md`, `config.yaml`, `permissions.yaml`.
    - Safety precedence: local configs are advisory and CANNOT override macros safety rules (role constraints, destructive‑git prohibitions, TPCP policy) unless explicitly allowed in `prompt.md`.
11. **RECOMMENDED WORKFLOW:**
    1. **OBS:** Scans and proposes a solution in `solutions/`.
    2. **USER:** Reviews proposal. Optionally updates `prompt.md` with feedback.
    3. **EXE:** Implements the approved plan.
    4. **USER:** Verifies changes:
       - **Success:** User commits changes manually (`git commit`).
       - **Failure:** User reverts changes (`git checkout .`), updates `prompt.md` with error details, and restarts from **OBS** step.
12. **SESSION START:** Every new task or session MUST begin with running `upd` (to sync git state) and/or `prompt` (to ingest the latest instructions from `prompt.md`). This ensures the agent is not working on stale data.
13. **MACRO RESOLUTION POLICY (NO CACHED MACROS):**
    - For commands `tellmeobs`, `tellmeexe`, `prompt`, `upd`, `cleanup`, you MUST load and follow the definitions from the root `macros.md` file at execution time.
    - Do NOT use any built‑in, hardcoded, or cached versions of these macros. If a previous in‑memory definition conflicts with the root file, discard it and reload from disk.
    - If `macros.md` is missing or unreadable, STOP and inform the user to restore the file. No fallback behavior is allowed.
     - Log the detected `Version:` line from `macros.md` at the start of each macro run.

14.1 **CONFIG DISCOVERY LOGGING:**
    - At macro start, if `CONFIG_DIR` is set per Rule 10, log: `Config: <CONFIG_DIR> detected (AGENTS.md, config.yaml, permissions.yaml)`

15. **ROLE PRIORITY & PROMPT CONFLICTS:**
    - If `prompt.md` or a direct user request instructs an agent to perform actions that conflict with its current role (e.g., an OBSERVER is asked to "fix" or "create" files outside `/AGENTS/`), the agent MUST prioritize its Role Constraints.
    - In such cases, the OBSERVER agent MUST only propose changes within a solution file in its `solutions/` directory, and NOT execute them directly.
    - The agent should briefly notify the user of this priority in the chat to avoid confusion.

16. **IMPLEMENTATION ACK PROTOCOL (IAP):**
    - Purpose: provide a reliable, machine-detectable confirmation that an OBS proposal was implemented by an EXE agent.
    - EXE MUST create an acknowledgement file (ACK) for each implemented proposal at:
      `/AGENTS/<NAME>-EXE/solutions/ack/<proposal_id>.ack`
    - ACK file format (plain text, one field per line):
      `ID: <proposal_id>`
      `IMPLEMENTED_AT: <timestamp>`
      `EXECUTOR: <NAME>-EXE`
      `FROM_FILE: <path_to_observer_proposal>`
      `CHANGES_REF: <path_to_exe_log>#L<line>`
    - Folders MUST be created if missing. EXE writes only inside its own folder.

17. **FINAL COMMIT+PUSH POLICY:**
    - Цель: предлагать commit/push только в конце работы, после завершения всех изменений.
    - По умолчанию: ASK — final commit/push выполняется ТОЛЬКО после явного согласия пользователя в чате.
    - Final (после всех изменений):
      * ASK USER: «Выполнить final commit+push сейчас?»
        - Yes → `git add -A`; `git commit -m "exe:final <YYYYMMDD_HHMMSS> — <summary>"`; если нет `PUSH_MODE: FINAL_MANUAL` → `git push origin HEAD`
        - No  → пропустить; оставить результаты локально без коммита/пуша
    - Переопределения через `prompt.md` (по желанию пользователя):
      * `PUSH_MODE: AUTO` — разрешить final commit+push без дополнительного вопроса.
      * `PUSH_MODE: FINAL_MANUAL` — при согласии на финализацию выполнить только commit (без push).
    - Ограничения: не предлагать промежуточные savepoint commit/push, если пользователь не попросил об этом явно.
18. **SEARCH POLICY (MANDATORY):**
    - Before answering any user request or acting on an instruction, run a quick repository search for relevant keywords from the latest user message.
    - Preferred: `rg -n -S "<keywords>"` from repo root.
    - Fallback: `grep -RIn --no-heading "<keywords>" .`
    - Exclude folders listed in Rule #5.

19. **NO SKILL TOOL FOR MACROS:**
    - Commands `tellmeobs`, `tellmeexe`, `prompt`, `upd`, `cleanup` are macros defined in this file. They are NOT Claude Code skills.
    - When any of these commands is received, the agent MUST NOT invoke the `Skill` tool under any circumstance — even if a skill with a matching or similar name (e.g. `tellme`) exists in the skills registry.
    - The correct execution path: read `macros.md` from disk → follow the matching macro section → execute steps.
    - Rationale: the `Skill` tool resolves by prefix/substring match and can incorrectly trigger on `tellmeobs`/`tellmeexe` when a `tellme` skill exists, producing wrong behavior.

=====================================================================
MACRO: prompt
=====================================================================

When user types: prompt

Execute:
0) Early Identity & Local Config Preload (ConfigFirst):
   - Detect `<NAME>` (CLAUDE/CODEX/KIMI/GEMINI) per Global Rule #2.
   - If detection fails → STOP with error (do not continue).
   - If config dir for `<NAME>` exists (see Global Rule #10) → read it first; log `Boot: ConfigFirst <CONFIG_DIR>`.
1) Macro Source Check:
   - Locate and read `./macros.md` at repository root. If missing/unreadable: STOP with error.
   - Record `Version:` in logs. Discard any cached macro definitions.
2) Role Check:
   - Check if an agent folder (/AGENTS/<NAME>-OBS/ or /AGENTS/<NAME>-EXE/) is already active in this session.
   - If NO active role:
     ASK USER: "No active role detected. Should I process this prompt as OBSERVER (audit/scan) or EXECUTOR (implementation)?"
     Wait for answer, then run `tellmeobs` or `tellmeexe` first.
3) Read Instruction:
   - Read `prompt.md` from the root directory.
   - Summarize the task.
4) Sync to Context:
   - Update /AGENTS/<NAME>_CONTEXT.md with the content of `prompt.md` under a "Current Task" header.
5) Execute Role Logic:
   - If OBS: Perform Deep Scan focusing on the requirements in `prompt.md`.
   - If EXE: Include requirements from `prompt.md` into the Master Plan.

=====================================================================
MACRO: tellmeobs
=====================================================================

When user types: tellmeobs

Execute the following:

0) Early Identity & Local Config Preload (ConfigFirst):
   - Detect `<NAME>` (CLAUDE/CODEX/KIMI/GEMINI) per Global Rule #2.
   - If detection fails → STOP (no folders/logs should be created).
   - If config dir for `<NAME>` exists (see Global Rule #10) → read it first; log `Boot: ConfigFirst <CONFIG_DIR>`.
1) Macro Source Check:
   - Locate and read `./macros.md` at repository root. If missing/unreadable: STOP with error.
   - Record `Version:` in logs. Discard any cached macro definitions.
2) Identity
   - Detect your actual model (CLAUDE, CODEX, KIMI, GEMINI) per Global Rule #2. Do **not** continue if you cannot match one of these names.
   - If detection fails, stop and inform the user about the unknown identity (no folders/logs should be created).
   - Once determined, set `<NAME>` to that value, announce `Agent ID: <NAME>-OBS`, and use `/AGENTS/<NAME>-OBS/` as the workdir.

1.1) Local Config (note)
   - Local config for `<NAME>` has ALREADY been read in step 0 (ConfigFirst).
   - Log detection as `Config: <CONFIG_DIR> detected` (or `Config: NONE`).

3) Environment
   Create if missing:
     /AGENTS/
     /AGENTS/<NAME>-OBS/
     /AGENTS/<NAME>-OBS/logs/
     /AGENTS/<NAME>-OBS/solutions/
     /AGENTS/<NAME>-OBS/instructions/

4) Registry & Context
   - In /AGENTS/AGENTS.md:
     Add or update only your own line:
     Agent: <NAME>-OBS | Role: OBSERVER | Status: COLLECTING_DATA | LastActive: <timestamp>

   - Context Read:
     Read ALL files matching /AGENTS/*_CONTEXT.md and the root `prompt.md`.

    - Context Initialization:
      If /AGENTS/<NAME>_CONTEXT.md is missing, create it.

4.1) Prompt Guard (EMPTY PROMPT baseline)
   - Determine if root `prompt.md` is EMPTY:
     Conditions (any true → EMPTY):
       * File missing OR zero-length after trim
       * Contains the cleanup stub text (e.g., heading "# Prompt" and the phrase "No active task")
   - If EMPTY:
     BASELINE OBSERVER FLOW (no proposals yet):
       1) Perform a broad repo overview scan (top-level folders/files; note key areas like `cv_cases_(projects)/` and `/AGENTS/`).
       2) Update `/AGENTS/<NAME>_CONTEXT.md` with sections:
          - "Baseline Mode: PROMPT_EMPTY"
          - "Repo Overview" (structure highlights)
          - "Next Steps" (ask user to populate `prompt.md`)
       3) Log baseline init in `/AGENTS/<NAME>-OBS/logs/session_YYYYMMDD_HHMMSS.log`.
       4) STOP here (skip Deliverable/PROPOSAL generation until a non-empty prompt is provided).

5) Strict Constraint
   - You MUST NOT modify any project files outside your Workdir.
   - Exception: You may update /AGENTS/AGENTS.md and /AGENTS/<NAME>_CONTEXT.md.
   - You MUST NOT run destructive git commands.
   - Allowed: read_file, grep, ls, codebase_investigator.
   - Allowed write ONLY inside your own folder and the two files mentioned above.

6) Deep Scan
   Use 'codebase_investigator' and other tools to scan:
     - project source directories
     - /AGENTS/*/solutions/ (to see what other agents proposed)
     - /AGENTS/<NAME>-OBS/instructions/ (for custom rules)
     - Root `prompt.md` for specific manual tasks.
    Exclude global ignore list above.

   Implementation Phase Awareness:
   - BEFORE proposing changes, explicitly check for any executor folders:
     /AGENTS/*-EXE/
   - If any exist, record "Implementation Phase: ACTIVE" in your context and avoid duplicate/conflicting proposals.

7) Context Update
   Update /AGENTS/<NAME>_CONTEXT.md with:
   - Current Goal/North Star (incorporating `prompt.md`).
   - Key findings from the current scan.
   - Summary of the project state as you see it.

8) Deliverable
   Create file:
    /AGENTS/<NAME>-OBS/solutions/YYYYMMDD_<slug>_NNN_fix_description.md

   Content must include:

   ID: <NAME>-OBS-YYYYMMDD-NNN
    STATUS: PROPOSED
   CREATED: <timestamp>
   SCOPE: <affected paths>

   PROBLEM:
   <clear description>

   EVIDENCE:
   <findings>

   PROPOSED FIX:
   <technical solution>

   RISK/ROLLBACK:
   <risk assessment>

9) Logging
   Append to:
   /AGENTS/<NAME>-OBS/logs/session_YYYYMMDD_HHMMSS.log

   First line:
   [HH:mm:ss] | TYPE: Init | DESCRIPTION: Observer mode active for <NAME>.

=====================================================================
MACRO: tellmeexe
=====================================================================

When user types: tellmeexe

Execute:

0) Early Identity & Local Config Preload (ConfigFirst):
   - Detect `<NAME>` (CLAUDE/CODEX/KIMI/GEMINI) per Global Rule #2.
   - If detection fails or outside allowed list → STOP.
   - If config dir for `<NAME>` exists (see Global Rule #10) → read it first; log `Boot: ConfigFirst <CONFIG_DIR>`.
1) Macro Source Check:
   - Locate and read `./macros.md` at repository root. If missing/unreadable: STOP with error.
   - Record `Version:` in logs. Discard any cached macro definitions.
2) Identity
   - Detect your actual model (CLAUDE, CODEX, KIMI, GEMINI) per Global Rule #2 before doing anything else.
   - If detection fails or yields a name outside the allowed list, stop and notify the user instead of continuing.
   - Once confirmed, set `<NAME>` accordingly, announce `Agent ID: <NAME>-EXE`, and operate inside `/AGENTS/<NAME>-EXE/`.

1.1) Local Config (note)
   - Local config for `<NAME>` has ALREADY been read in step 0 (ConfigFirst).
   - Log detection as `Config: <CONFIG_DIR> detected` (or `Config: NONE`).

3) Environment
   Create if missing:
     /AGENTS/<NAME>-EXE/
     /AGENTS/<NAME>-EXE/logs/
     /AGENTS/<NAME>-EXE/solutions/

4) Registry & Context
   - Update only your own line in /AGENTS/AGENTS.md:
     Agent: <NAME>-EXE | Role: EXECUTOR | Status: READY_TO_DEPLOY | LastActive: <timestamp>
    - Read ALL /AGENTS/*_CONTEXT.md and the root `prompt.md` to ensure the global strategy is understood.

4.1) Prompt Guard (EMPTY PROMPT baseline)
   - Determine if root `prompt.md` is EMPTY (same rules as in tellmeobs).
   - If EMPTY:
     BASELINE EXECUTOR CHECK (no plan/implementation):
       1) Run macro `upd` to sync state.
       2) Inspect `/AGENTS/*-OBS/solutions/` for files containing `STATUS: PROPOSED` and list them.
       3) Collect the current git working tree summary.
       4) Write a short report to `/AGENTS/<NAME>-EXE/solutions/baseline_exe_report_YYYYMMDD_HHMMSS.md` with:
          - HEADER (Agent, timestamp, PROMPT: EMPTY)
          - Observed proposals
          - Git working tree summary
          - Next Steps: "Please update prompt.md with the next task."
       5) Append a log line to `/AGENTS/<NAME>-EXE/logs/implement_YYYYMMDD_HHMMSS.log` with TYPE: Baseline.
       6) STOP here (skip Master Plan and any modifications until a non-empty prompt is provided).

5) Vacuum Mode
   Scan ALL observer folders:
   /AGENTS/*-OBS/solutions/
   Collect files containing:
   STATUS: PROPOSED

6) Master Plan
   Create:
   /AGENTS/<NAME>-EXE/solutions/master_plan.md

   The master_plan.md MUST be DETAILED and ACTIONABLE so the user can
   review it without opening each observer proposal. The document MUST
   follow this structure:

   HEADER
   - Agent: <NAME>-EXE
   - CREATED: <timestamp>
   - SOURCE_PROMPT: short one-line summary from `prompt.md`
   - SIGNOFF: PENDING (set to APPROVED after user confirmation)

   SUMMARY OF GOALS
   - High-level objectives derived from `prompt.md` and context files.

   PROPOSALS INCLUDED
   For each included proposal:
   - INCLUDE: <proposal_path>
   - ID: <proposal_id>
   - SCOPE: paths parsed from the proposal
   - CONFLICT CHECK: state any conflicts with /AGENTS/*_CONTEXT.md or `prompt.md`

   IMPLEMENTATION PLAN (REQUIRED)
   For each included proposal, provide a concrete, per-file action plan:
   - AFFECTED PATHS: explicit list of files/dirs to change
   - ACTIONS: ordered steps with type + path + description
     * ADD | UPDATE | MOVE | DELETE: <path>
       - RATIONALE: <why this is needed>
       - CONTENT SUMMARY: <what will be added/changed>
       - PRECONDITION: <checks to run before>
       - POSTCONDITION: <expected state after>
   - ACCEPTANCE CRITERIA: checklist of verifiable outcomes
   - VALIDATION STEPS: exact commands/checks (e.g., `rg` patterns) the user can run
   - RISK/ROLLBACK: concise risk and how to revert
   - DEPENDENCIES: other files or proposals this depends on
   - OUT-OF-SCOPE: what will NOT be changed

   BATCHING PLAN
   - Group actions into batches (e.g., by directory), with rough priority and ETA (t-shirt sizing OK).

   GLOBAL IMPLEMENTATION CHECKLIST
   - Single checklist aggregating all actions across proposals.

   NOTES
   - Do not proceed with implementation until the user confirms the master plan (chat approval).
   - After approval, reference the exact action labels from this plan in implementation logs.

7) Implementation Rules
   Before ANY modification:
     Run macro: upd

   If git status shows uncommitted changes:
     Log the state, but DO NOT stop implementation solely because the tree is dirty.

   Never modify ANY folder inside /AGENTS/ other than your own.

8) Logging
   Append to:
   /AGENTS/<NAME>-EXE/logs/implement_YYYYMMDD_HHMMSS.log

   Format:
   [HH:mm:ss] | TYPE: Implementation | FROM_ID: <proposal_id> | FROM_FILE: <proposal_path> | FILE: <changed_path> | DESCRIPTION: <summary>

9) Acknowledgement Files (IAP)
   For each implemented proposal (identified by FROM_ID):
     - Ensure folder exists: `/AGENTS/<NAME>-EXE/solutions/ack/`
     - Create or update file: `/AGENTS/<NAME>-EXE/solutions/ack/<proposal_id>.ack`
       Content (plain text):
         ID: <proposal_id>
         IMPLEMENTED_AT: <timestamp>
         EXECUTOR: <NAME>-EXE
         FROM_FILE: <proposal_path>
         CHANGES_REF: /AGENTS/<NAME>-EXE/logs/implement_YYYYMMDD_HHMMSS.log#L<line>

10) Finalization (commit+push only at the end, только с согласия пользователя)
   - ASK USER: «Выполнить final commit+push сейчас?»
     * If Yes:
       - Stage all changes: `git add -A`
       - Commit: `git commit -m "exe:final <timestamp> — <summary>"`
       - Push: run `git push origin HEAD` unless root `prompt.md` contains `PUSH_MODE: FINAL_MANUAL`
     * If No: skip commit/push (оставить изменения локально для ручной проверки)

=====================================================================
MACRO: upd
=====================================================================

Execute:

0) Early Identity & Local Config Preload (ConfigFirst):
   - Detect `<NAME>` (CLAUDE/CODEX/KIMI/GEMINI) per Global Rule #2.
   - If detection fails → STOP.
   - If config dir for `<NAME>` exists (see Global Rule #10) → read it first; log `Boot: ConfigFirst <CONFIG_DIR>`.
1) Macro Source Check:
   - Locate and read `./macros.md` at repository root. If missing/unreadable: STOP with error.
   - Record `Version:` in logs. Discard any cached macro definitions.
2) Project Sync
   Run:
     git status
     git diff --name-only
     ls -d AGENTS/*/  # Force discovery of active agent modules

3) Dirty Tree Note
   If uncommitted changes detected:
     - OBS: log only.
     - EXE: log only. Do NOT stop implementation solely because the tree is dirty.

4) Registry Sync
   Update only your own LastActive in /AGENTS/AGENTS.md

5) Proposal Sync
   OBS:
     - For each of your proposals in `/AGENTS/<NAME>-OBS/solutions/`:
       1) Read `ID: <proposal_id>`.
       2) Check for ACK (IAP): `/AGENTS/*-EXE/solutions/ack/<proposal_id>.ack`.
       3) If no ACK, fallback: search EXE logs `/AGENTS/*-EXE/logs/implement_*.log` for `FROM_ID: <proposal_id>`.
       4) If ACK or log evidence found:
          - Update your proposal file:
            * Replace `STATUS: PROPOSED` -> `STATUS: IMPLEMENTED`
            * Append block at the end:
              `IMPLEMENTED: <timestamp>`
              `IMPLEMENTED_BY: <EXE_NAME>-EXE`
              `EVIDENCE: <ack_path or log_path#Lline>`
          - Append to your OBS log:
            `[HH:mm:ss] | TYPE: Sync | DESCRIPTION: My solution <proposal_id> was implemented by <EXE_NAME>-EXE`

   EXE:
     - Check for new files with `STATUS: PROPOSED` across `/AGENTS/*-OBS/solutions/` and collect into a processing list.
     - After implementing any proposal, you MUST create/update its ACK per Rule 16 (IAP).

6) Logging
   Append:
   [HH:mm:ss] | TYPE: Sync | DESCRIPTION: State synchronized across AGENTS/ directory.

=====================================================================
MACRO: cleanup
=====================================================================

When user types: cleanup

Execute:
0) Early Identity & Local Config Preload (ConfigFirst):
   - Detect `<NAME>` (CLAUDE/CODEX/KIMI/GEMINI) per Global Rule #2.
   - If detection fails → STOP.
   - If config dir for `<NAME>` exists (see Global Rule #10) → read it first; log `Boot: ConfigFirst <CONFIG_DIR>`.
1) Macro Source Check:
   - Locate and read `./macros.md` at repository root. If missing/unreadable: STOP with error.
   - Record `Version:` in logs. Discard any cached macro definitions.
2) Identification:
   Identify agent folders in /AGENTS/.
3) Wipe Logs & Solutions:
   - For each agent folder:
     - Remove all files in `logs/`
     - Remove all files in `solutions/`
   - Log the action in a fresh `logs/session_cleanup_<timestamp>.log`.
4) Reset Contexts (fresh session stubs):
   - For each existing `/AGENTS/*_CONTEXT.md` file:
     - Overwrite with a minimal template:
       ``
       # <NAME> CONTEXT

       Reset: <timestamp>

       ## Current Task
       (empty; ready for next session)

       ## Notes
       Use `prompt` or `tellmeobs`/`tellmeexe` to initialize a new session.
       ``
     - Keep filename and casing intact; update only the body.
   - Rationale: ensure no stale tasks bleed into the next run.
5) Reset Root Prompt (`prompt.md`):
   - Overwrite root `prompt.md` with a stub to avoid re-running old tasks:
     ``
     # Prompt

     Reset: <timestamp>

     No active task. Add new instructions here.
     ```
   - Rationale: ensures a clean start for the next session and prevents accidental repeat of previous operations.
6) Preserve:
   - DO NOT delete `*_CONTEXT.md` files (only reset them as above).
   - DO NOT delete any `instructions/` content.
7) Finalization Rule:
   - This macro is the concluding step after a batch of changes to ensure a clean development environment for the next session.
   - Postconditions (must be true):
     - All `/AGENTS/*/logs/` directories contain only the newly written `session_cleanup_<timestamp>.log`.
     - All `/AGENTS/*/solutions/` directories are empty.
     - All `/AGENTS/*_CONTEXT.md` files exist and contain the reset stub with the current timestamp.
     - Root `prompt.md` exists and contains the reset stub with the current timestamp.
8) Optional Commit & Push:
   - Stage and commit cleanup changes (AGENTS only). Example:
     - `git add AGENTS`
      - `git add prompt.md`
      - `git commit -m "cleanup(AGENTS): wipe logs/solutions; reset contexts and prompt.md for a fresh session"`
   - Respect Global Rule #9: only `git push` if explicitly approved by the user in chat.

=====================================================================
END OF MACROS
=====================================================================
