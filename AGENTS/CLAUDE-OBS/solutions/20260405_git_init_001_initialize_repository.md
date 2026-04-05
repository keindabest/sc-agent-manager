ID: CLAUDE-OBS-20260405-001
STATUS: PROPOSED
CREATED: 2026-04-05 21:20 +0100
SCOPE: /sc-agent-manager/ (root), .gitignore (new file)

PROBLEM:
Проект sc-agent-manager не является git-репозиторием. Директория содержит рабочие файлы мульти-агентной системы (macros.md, prompt.md, AGENTS/, конфиги агентов), но команда `git status` возвращает "fatal: not a git repository". Без git невозможно версионировать изменения, откатываться к предыдущим состояниям или публиковать проект (например, на GitHub/GitLab).

EVIDENCE:
- `git status` → "fatal: not a git repository (or any of the parent directories): .git"
- `ls -la` корневой директории: НЕТ папки `.git`
- Файлы в директории: .claude/, .codex/, .gemini/, AGENTS/, macros.md, prompt.md
- Нет .gitignore, нет README.md

PROPOSED FIX:

### Шаг 1 — Создать .gitignore
Создать файл `.gitignore` в корне проекта, чтобы исключить временные и чувствительные файлы:

```
# OS
.DS_Store
Thumbs.db

# Node / npm (если появятся)
node_modules/
npm-debug.log

# Logs агентов (опционально — если не нужно версионировать)
# AGENTS/*/logs/

# Временные файлы
*.tmp
*.swp
```

> **Примечание по AGENTS/logs/:** Логи агентов — это артефакты работы системы. Рекомендуется их включить в git для аудита, но пользователь может добавить `AGENTS/*/logs/` в .gitignore если они не нужны в истории.

### Шаг 2 — Инициализировать репозиторий

```bash
cd /path/to/sc-agent-manager
git init
git branch -M main
```

### Шаг 3 — Первый коммит

```bash
git add .
git commit -m "init: initialize sc-agent-manager multi-agent system"
```

### Шаг 4 (опционально) — Привязать удалённый репозиторий

Если пользователь хочет разместить на GitHub/GitLab:

```bash
git remote add origin <REMOTE_URL>
git push -u origin main
```

RISK/ROLLBACK:
- РИСК: Минимальный. `git init` создаёт только папку `.git`, не изменяет существующие файлы.
- РИСК: Если в директории есть чувствительные данные (токены, ключи) — они попадут в первый коммит. Убедиться, что .gitignore создан ДО `git add`.
- ОТКАТ: Удалить папку `.git` → `rm -rf .git` (репозиторий будет удалён, файлы останутся).
- ОТКАТ коммита: `git reset --soft HEAD~1` (отменить коммит, оставить файлы staged).
