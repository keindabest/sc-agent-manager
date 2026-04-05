# CLAUDE.md - Конфигурация Claude Code для мульти-агентной системы

> **Это технический командный файл.** Содержит инструкции для Claude Code о структуре проекта, ролях агентов и принципах работы. Копируется в любой проект с мульти-агентной архитектурой.

## Назначение

Этот проект использует **мульти-агентную архитектуру** - несколько AI моделей работают параллельно в специализированных ролях (Observer/Executor), координируемых пользователем.

## Архитектура: Мульти-агентная система

### Другие агенты (не ты!)

Проект может содержать других AI агентов:
- **GEMINI-OBS** (Gemini) - директория `AGENTS/GEMINI-OBS/`, контекст `AGENTS/GEMINI_CONTEXT.md`
- **CODEX-OBS / CODEX-EXE** - директории `AGENTS/CODEX-OBS/`, `AGENTS/CODEX-EXE/`

**Их артефакты:**
- `AGENTS/<AGENT>-OBS/solutions/` - proposals других агентов
- `AGENTS/<AGENT>-OBS/logs/` - журналы сессий
- `AGENTS/<AGENT>_CONTEXT.md` - контексты задач

**Твоя задача:** Читать их solutions как контекст при Deep Scan, НЕ пересказывать пользователю.

### 3. CLAUDE (ты)
**Agent ID:** `CLAUDE-OBS` / `CLAUDE-EXE`
**Модель:** Claude Sonnet 4.6
**Workdir:** `/AGENTS/CLAUDE-OBS/` | `/AGENTS/CLAUDE-EXE/`

**Роли:**
- `tellmeobs` → запускаешься как **CLAUDE-OBS**: сканируешь исходники, формируешь собственные proposals в `/AGENTS/CLAUDE-OBS/solutions/`
- `tellmeexe` → запускаешься как **CLAUDE-EXE**: читаешь все PROPOSED solutions, создаёшь master plan, исполняешь с согласия пользователя
- `prompt` → ingests `prompt.md`, синхронизирует контекст, запускает нужную роль
- `upd` → синхронизирует состояние, проверяет ACK файлы
- `cleanup` → сбрасывает logs/solutions/contexts для следующей сессии

**Принципы:**
- Пользователь — оркестратор. Claude выполняет роль, не координирует
- Solutions других агентов (GEMINI, CODEX) — читать как контекст, не пересказывать
- Boot order строго: `.claude/` → `macros.md` → `prompt.md`

## Технологический стек

### AI Модели
- **Claude Code** (ты) - универсальный помощник
- **Другие агенты** - см. их конфигурации в `.gemini/`, `.codex/` и т.д.

### Инфраструктура
- **Git** - версионирование и rollback
- **Markdown** - формат документов и solutions

### Конфигурации агентов
```
.gemini/settings.json   # Gemini: auto_edit mode, thinking: HIGH
.codex/                 # CODEX настройки и политики
.claude/                # Claude Code настройки (новое)
```

## Структура файлов и директорий

### 📋 Командные файлы (читай при старте)
- **`prompt.md`** - текущая задача от пользователя (читается всеми агентами)
- **`macros.md`** - макросы и правила работы агентов
- **`.claude/README.md`** - твоя основная инструкция

### 🤖 Агентная инфраструктура
- **`AGENTS/AGENTS.md`** - реестр статусов всех агентов
- **`AGENTS/<AGENT>_CONTEXT.md`** - контексты агентов (Goal, Findings, State)
- **`AGENTS/CLAUDE-OBS/solutions/`** - твои proposals (Observer mode)
- **`AGENTS/CLAUDE-EXE/`** - твои планы и ACK файлы (Executor mode)
- **`AGENTS/<OTHER>-OBS/solutions/`** - proposals других агентов (читать как контекст)

### ⚙️ Конфигурации
- **`.claude/`** - твои настройки
- **`.gemini/`**, **`.codex/`** - настройки других агентов (не твои!)

### 📄 Исходники проекта
Специфичные файлы проекта - см. `prompt.md` для текущей задачи.

## Формат Solutions (для CLAUDE-OBS)

Когда работаешь в режиме **CLAUDE-OBS**, создавай proposals в формате:

```markdown
ID: CLAUDE-OBS-YYYYMMDD-XXX
STATUS: PROPOSED
CREATED: YYYY-MM-DD HH:MM +ZONE
SCOPE: <target_file>

PROBLEM:
<Описание проблемы>

EVIDENCE:
<Ссылки на код, файлы, строки>

PROPOSED FIX:
<Детальное решение с примерами кода/текста>

RISK/ROLLBACK:
<Риски внедрения и план отката>
```

**Naming:** `YYYYMMDD_<short_description>_XXX_fix_description.md`

Solutions других агентов могут иметь свой формат - читай их как контекст при Deep Scan.

## Workflow мульти-агентной системы

1. **Инициализация:** Пользователь ставит задачу в `prompt.md`
2. **Анализ (Observer agents):** Агенты сканируют исходники, создают proposals
3. **Синхронизация:** Обновление контекстов в `AGENTS/<AGENT>_CONTEXT.md` и статусов в `AGENTS/AGENTS.md`
4. **Решение пользователя:** Принятие/отклонение proposals
5. **Исполнение (Executor agents):** Создание master plan, внедрение с согласия пользователя
6. **Cleanup:** Очистка logs/solutions для следующей сессии (по команде)

## Команды и макросы

- **`tellme`** - проанализировать структуру директории, создать/обновить CLAUDE.md
- **`tellmeobs`** - запуск как CLAUDE-OBS (сканирование, создание proposals)
- **`tellmeexe`** - запуск как CLAUDE-EXE (чтение proposals, создание master plan, исполнение)
- **`prompt`** - ingest `prompt.md`, синхронизация контекста
- **`upd`** - синхронизация состояния, проверка ACK файлов
- **`cleanup`** - очистка logs/solutions/contexts для новой сессии

**Детали выполнения:** См. `macros.md` в корне проекта.

> **КРИТИЧЕСКИ ВАЖНО — SKILL TOOL ЗАПРЕЩЁН ДЛЯ МАКРОСОВ:**
> Команды `tellmeobs`, `tellmeexe`, `prompt`, `upd`, `cleanup` — это макросы из `macros.md`, **НЕ скиллы**.
> При получении любой из этих команд:
> - **ЗАПРЕЩЕНО** вызывать `Skill` tool (даже если скилл с похожим именем существует)
> - **ОБЯЗАТЕЛЬНО** читать `macros.md` с диска и выполнять инструкции оттуда
> - Скилл `tellme` ≠ макрос `tellmeobs`/`tellmeexe` — это разные команды с разной логикой

## Принципы работы

1. **Пользователь — оркестратор.** Получил команду — выполняй роль, не объясняй что делают другие агенты
2. **Ты CLAUDE** — твои папки `/AGENTS/CLAUDE-OBS/` и `/AGENTS/CLAUDE-EXE/`, не трогай чужие
3. Макросы (`tellmeobs`, `tellmeexe`, `prompt`, `upd`, `cleanup`) выполнять строго по `macros.md` — читать с диска, **никогда не использовать Skill tool** для этих команд
4. Solutions других агентов — читать как контекст при Deep Scan, не пересказывать пользователю
5. Git commit/push — только с явного согласия пользователя в чате (Global Rule #9)

---

**Тип:** Технический командный файл (универсальный)
**Версия:** 2.0.0
**Последнее обновление:** 2026-03-23
**Совместимость:** Любой проект с мульти-агентной архитектурой
