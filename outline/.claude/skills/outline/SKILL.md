---
name: outline
description: Create presentation outlines and structures using agent pipelines with presets, review, and iterative refinement.
argument-hint: "[--help | --preset <name> | --format <fmt> | --questions | --new-preset | --edit <file> <comment> | --review <file> [--preset <name>] | --learn=N [preset] | --deep_learn=N [preset]] [<topic>]"
---

# Outline — Presentation Structure Generator

You generate presentation structures using agent pipelines. Each pipeline consists of specialized agents (generator, reviewer, fixer, etc.) that iteratively create, review, and refine a presentation outline.

## Language Rule

**CRITICAL:** All user-facing output MUST be in Russian. This includes:
- Help text, prompts, error messages, reports, summaries
- Generated presentation structures (slide titles, bullet points, speaker notes)
- Wizard questions and confirmations
- Agent-generated content (structures, reviews, feedback)

Internal skill logic, variable names, file names, and YAML fields remain in English.

## References

Before executing, internalize these references:
- `references/preset-format.md` — **CRITICAL**: Preset directory specification (metadata, storage, naming, collisions)
- `references/pipeline-format.md` — **CRITICAL**: Pipeline execution specification (steps, loops, reviewer protocol, context variables)
- `references/output-formats.md` — Output format specs (slidev, universal, custom)

## Input Parsing

Parse the user's input to determine the subcommand or mode:

### Subcommands (handle before anything else)

**`--help`**: Display usage help and stop. Show:

```
Outline — Генератор структуры презентаций

Использование:
  /outline <тема>                               Сгенерировать структуру (авто-выбор пресета)
  /outline --preset <имя> <тема>                Использовать конкретный пресет
  /outline --format <slidev|universal|custom>   Переопределить формат вывода
  /outline --questions <тема>                   Генерация с уточняющими вопросами
  /outline --edit <файл> <комментарий>           Отредактировать существующую структуру
  /outline --review <файл> [--preset <имя>]     Ревью и улучшение любой структуры
  /outline --new-preset                         Создать новый пресет агентного пайплайна
  /outline --learn=N [пресет]                   Обучить агентов на N тестовых прогонах
  /outline --deep_learn=N [пресет]             Итеративное обучение: N циклов (прогон → анализ → правки)
  /outline --help                               Эта справка

Как это работает:
  Outline использует агентные пайплайны для итеративного создания, рецензирования
  и улучшения структур презентаций. Каждый пресет — это набор специализированных
  агентов (генератор, рецензент, редактор и т.д.), настроенных под конкретный тип
  презентации.

  Если пресет не указан, скилл автоматически выбирает подходящий
  или использует агентов по умолчанию (универсальный генератор + рецензент).

Форматы вывода:
  slidev      (по умолчанию) ## Слайд N: Заголовок + буллиты — совместим с /slidev
  universal   Секции, тезисы, заметки спикера — не привязан к инструменту
  custom      Определяется пресетом

Хранение пресетов:
  Локально:  .outline-presets/<имя>/
  Глобально: ~/.claude/outline-presets/<имя>/
  Поиск:     сначала локально, потом глобально
```

Stop here — do not proceed to generation.

---

**`--new-preset`**: Run the Create Preset Procedure (CP-1 through CP-8). Stop here — do not proceed to generation.

---

**`--edit <file> <comment>`**: Run the Edit Procedure (E-1 through E-5). The file must be a previously generated outline with metadata frontmatter. Stop here — do not proceed to generation.

---

**`--review <file> [--preset <name>]`**: Run the Review Procedure (R-1 through R-6). Works with any structure file — no metadata required. Stop here — do not proceed to generation.

---

**`--learn=N [preset]`**: Run the Learn Procedure (L-1 through L-6). Parse N from the argument (e.g., `--learn=5`). Optional `preset` argument specifies which preset to train. Stop here — do not proceed to generation.

---

**`--deep_learn=N [preset]`**: Run the Deep Learn Procedure (DL-1 through DL-7). Parse N from the argument (e.g., `--deep_learn=5`). N is the number of training cycles. Optional `preset` argument specifies which preset to train. Stop here — do not proceed to generation.

---

**Otherwise** — this is a generation request. Parse `--preset`, `--format`, `--questions`, and extract the topic. Run the Generate Procedure (G-1 through G-7).

---

## Generate Procedure

### G-1: Parse Command

Extract from the user's input:
- **Topic** — the presentation subject (everything that isn't a flag)
- **`--preset <name>`** — optional, specific preset to use
- **`--format <slidev|universal|custom>`** — optional, output format override
- **`--questions`** — optional flag, enables interactive Q&A mode (see below)

If no topic is provided, ask the user: "О чём будет презентация?"

### G-2: Select Preset

Three paths, in priority order:

**Path A: `--preset <name>` specified**

1. Look up `<name>` in local storage: `.outline-presets/<name>/`
2. If not found, look up in global storage: `~/.claude/outline-presets/<name>/`
3. If not found in either, error:
   ```
   Пресет '<name>' не найден.
   Искали в: .outline-presets/<name>/, ~/.claude/outline-presets/<name>/
   Доступные пресеты: <list all found presets, or "нет">
   ```
   Stop here.

**Path B: Auto-select from available presets**

1. Scan for presets in both locations:
   - `.outline-presets/*/preset.md`
   - `~/.claude/outline-presets/*/preset.md`
2. If presets exist, read each `preset.md` to extract `name`, `description`, `keywords`
3. Make a single LLM call (use the Agent tool) with the user's topic and all preset metadata:

   ```
   Given the presentation topic: "<topic>"

   Available presets:
   1. <name>: <description> (keywords: <keywords>)
   2. <name>: <description> (keywords: <keywords>)
   ...

   Which preset best matches this topic? Reply with ONLY the preset name,
   or "none" if no preset is a good match.
   ```

4. If the agent responds with a preset name → use that preset
5. If the agent responds "none" → use default (Path C)

**Path C: Use built-in default**

Use the default preset from `assets/default/` within the skill directory. This path is used when:
- No presets exist in local or global storage
- Auto-selection returns "none"

### G-3: Load & Validate Pipeline

1. Read the preset directory:
   - `preset.md` — extract frontmatter fields
   - `pipeline.md` — extract `steps` array
   - `agents/*.md` — read all agent files

2. **Validate pipeline:**
   - Every `agent` value in `steps` must have a corresponding file in `agents/`
   - Every `loop_with` value must reference an agent in `steps`
   - `stop_when` is required when `loop_with` is present
   - At least one `role: create` agent exists
   - At least one `role: review` agent exists

   If validation fails:
   ```
   Ошибка валидации пайплайна: <specific issue>
   Пример: Агент 'investor-reviewer' указан в pipeline.md, но файл agents/investor-reviewer.md не найден.
   ```
   Stop here.

3. **Determine active settings:**
   - `max_iterations` — from preset.md (default: 3)
   - **Active format** — `--format` flag > preset `format` field > `slidev`
   - Build `{{output_format}}` variable based on active format:
     - `slidev` → `"Используй формат slidev-аутлайна: ## Slide N: Заголовок, затем буллиты. Целься на 8-12 слайдов. Весь контент на русском языке."`
     - `universal` → `"Используй универсальный формат: ## Section N: Заголовок, в каждой секции — Тезис, Ключевые пункты буллитами, Заметки спикера в блок-цитатах. Весь контент на русском языке."`
     - `custom` → use the literal `custom_format_description` text from preset.md

### G-4: Run Pipeline Cycle

Initialize context variables:
```
topic         = user's topic
current_draft = "" (empty initially)
feedback      = "" (empty initially)
iteration     = 1
output_format = <constructed in G-3>
questions_mode = true if --questions flag was set, false otherwise
```

#### Interactive Q&A Mode (`--questions`)

When `questions_mode` is active, **each agent** (generator, fixer, reviewer) may pause to ask the user clarifying questions before producing output. This applies to every agent invocation throughout the pipeline — initial generation, re-runs after feedback, and review passes.

**How it works:**

When dispatching an agent via the Agent tool, append this instruction to the agent's substituted prompt:

```
INTERACTIVE MODE: Before producing your output, consider whether you need
any clarifications from the user to do a better job. If you have questions,
output ONLY a block starting with QUESTIONS: followed by numbered questions
(1-3 max, concise). If you have no questions, produce your output as normal.
```

After receiving the agent's response:
1. If the response starts with `QUESTIONS:` — extract the questions, present them to the user, wait for answers, then re-dispatch the same agent with the answers appended to its prompt as `User answers: <answers>`
2. If the response does NOT start with `QUESTIONS:` — proceed as normal (agent had no questions)

An agent may ask questions **at most once per invocation** — after receiving answers, it must produce its output.

**CRITICAL:** When `questions_mode` is false (default), do NOT append the interactive instruction. Agents run fully autonomously with no user interaction.

The `--questions` flag is also compatible with `--edit` and `--review` — when combined, agents in those procedures follow the same interactive Q&A protocol.

**Execute pipeline steps in order:**

For each step in the `steps` array:

#### Step with `role: create` (Generator)

1. Read the agent file from `agents/<agent>.md`
2. Substitute all `{{variables}}` in the agent prompt with current context values
3. Dispatch via the **Agent tool** as a subagent:
   - Provide the substituted prompt as the agent's task
   - Instruct the agent to output ONLY the structure, no preamble
4. Collect the agent's output → set `current_draft` to this output

#### Step with `role: review` + `loop_with` (Reviewer in a loop)

This step initiates the review→fix cycle:

1. Read the reviewer agent file from `agents/<agent>.md`
2. Substitute `{{variables}}` (including `{{current_draft}}`)
3. Dispatch the reviewer via the **Agent tool**
4. Parse the reviewer's output:
   - Find the last non-empty line
   - If it starts with `APPROVED` → exit loop, proceed to post-loop steps
   - If it starts with `NEEDS_REVISION:` → extract feedback text after the prefix

5. If `NEEDS_REVISION` and `iteration < max_iterations`:
   - Increment `iteration`
   - Set `feedback` to the extracted feedback text
   - Look up the `loop_with` agent:
     - If it has `role: fix` → read fixer agent, substitute variables, dispatch, collect output → update `current_draft`
     - If it has `role: create` (no-fixer pattern) → re-run the generator with feedback in context → update `current_draft`
   - Go back to step 1 (re-run reviewer)

6. If `NEEDS_REVISION` and `iteration >= max_iterations`:
   - Log: "Достигнут лимит итераций (N). Используем последний черновик."
   - Exit loop with current draft

#### Step with `role: review` WITHOUT `loop_with`

Run the reviewer once as a standalone evaluation. The output is informational only (no looping).

#### Step with `role: enhance` + `run_after: loop`

Skip during the main loop. These are handled in G-5.

### G-5: Run Optional Post-Loop Agents

After the main loop completes, run any steps with `run_after: loop`:

1. Read the agent file
2. Substitute `{{variables}}` (including final `current_draft`)
3. Dispatch via Agent tool
4. Append the agent's output to `current_draft` (separated by a blank line and `---`)

If an optional agent fails (`optional: true`), log the failure and continue. Do not abort.

### G-6: Save Output

**Topic slug rules:**
1. Lowercase the topic
2. Replace spaces with hyphens
3. Strip all non-alphanumeric characters except hyphens
4. Collapse consecutive hyphens into one
5. Truncate to max 50 characters at a word boundary (don't cut mid-word)

**File naming:**
- Primary: `outline-<topic-slug>.md`
- If file exists: `outline-<topic-slug>-2.md`, `outline-<topic-slug>-3.md`, etc.

**Save location:** Current working directory.

**Metadata frontmatter:** Prepend a YAML frontmatter block to the output file with generation metadata. This is used by `--edit` to locate the original preset and agents.

```yaml
---
topic: "<original topic>"
preset: "<preset name>"
preset_location: "<local|global|default>"
format: "<active format>"
---
```

- `preset_location`: `local` if from `.outline-presets/`, `global` if from `~/.claude/outline-presets/`, `default` if built-in default was used
- The frontmatter is followed by a blank line, then the structure content

Write the frontmatter + `current_draft` to the output file.

### G-7: Report

Print a summary:
```
Структура презентации готова.

  Пресет:     <preset name or "по умолчанию">
  Формат:     <active format>
  Итерации:   <number of iterations used> / <max_iterations>
  Файл:       <file path>

Для использования с /slidev (если формат slidev):
  /slidev <file path>
```

---

## Create Preset Procedure

### CP-1: Ask Preset Name

Ask the user:
```
Как назвать пресет? (kebab-case, например: investor-pitch, tech-talk)
```

Validate: must be kebab-case (lowercase letters, numbers, hyphens only). If invalid, explain and ask again.

### CP-2: Ask Description

Ask the user:
```
Для чего этот пресет? (краткое описание)
```

### CP-3: Ask Target Audience

Ask the user:
```
Кто целевая аудитория? (например: инвесторы, студенты, команда разработки, широкая публика)
```

### CP-4: Ask Output Format

Ask the user:
```
Формат вывода?
  1. slidev — ## Слайд N: Заголовок + буллиты (совместим с /slidev)
  2. universal — Секции, тезисы, заметки спикера
  3. custom — Вы опишете формат сами

Выбор (1/2/3):
```

If the user chooses "custom" (3), follow up:
```
Опишите формат, в котором генератор должен выдавать структуру:
```

### CP-5: Ask Storage Location

Ask the user:
```
Где сохранить пресет?
  1. глобально — ~/.claude/outline-presets/ (доступен везде)
  2. локально — .outline-presets/ (только в этом проекте)

Выбор (1/2):
```

### CP-6: Ask Additional Context (Optional)

Ask the user:
```
Дополнительный контекст или ограничения? (необязательно — нажмите Enter, чтобы пропустить)
```

### CP-6.5: Collision Check

Now that the storage location is known, check if a preset with this name already exists:

- If global: check `~/.claude/outline-presets/<name>/`
- If local: check `.outline-presets/<name>/`

**CRITICAL:** A preset with the same name in the OTHER location is NOT a collision.

If collision detected:
```
Пресет с именем '<name>' уже существует: <path>

  1. Перезаписать — заменить существующий пресет
  2. Переименовать — выбрать другое имя
  3. Отмена — прекратить создание

Выбор (1/2/3):
```

- Перезаписать → continue, files will be replaced
- Переименовать → go back to CP-1
- Отмена → stop here

### CP-7: Autonomous Pipeline Design

Using the collected information, autonomously design the agent pipeline. Do NOT ask the user for agent details — design everything yourself.

**Design decisions to make:**

1. **How many agents?** Minimum 2 (generator + reviewer). Add a fixer if the audience requires specialized revision logic. Add enhance agents if post-loop processing adds value (e.g., Q&A for educational content).

2. **Agent roles and prompts:** For each agent:
   - Name (kebab-case, descriptive)
   - Role (`create`, `review`, `fix`, `enhance`)
   - Input variables it needs
   - A detailed prompt tailored to the preset's audience, goals, and format

3. **Pipeline design:** Define the `steps` array in `pipeline.md`:
   - Execution order
   - Which agents loop together
   - Stop conditions
   - Optional post-loop agents

4. **Keywords:** Generate comprehensive keywords for `preset.md` based on the description and audience

**Write files to the storage location:**

```
<storage-path>/<name>/
  ├── preset.md
  ├── pipeline.md
  └── agents/
      ├── <agent-1>.md
      ├── <agent-2>.md
      └── ...
```

Use `max_iterations: 3` unless the preset type clearly benefits from more or fewer iterations.

### CP-8: Confirmation Summary

Print:
```
Пресет '<name>' создан.

  Путь:       <full path to preset directory>
  Агенты:     <list of agent names and roles>
  Пайплайн:   <brief flow description>
  Формат:     <format>

Использование:
  /outline --preset <name> <тема>

Или просто /outline <тема> — авто-выбор подберёт этот пресет,
если тема совпадёт с его ключевыми словами.
```

---

## Edit Procedure

### E-1: Parse Command

Extract from the user's input:
- **`<file>`** — path to an existing outline file
- **`<comment>`** — edit instruction (what to change)

If the file is missing or not provided, ask: "Какой файл отредактировать? Укажите путь к файлу outline-*.md"

If the comment is missing, ask: "Что изменить в структуре?"

### E-2: Read File and Extract Metadata

1. Read the file content
2. Parse the YAML frontmatter to extract `topic`, `preset`, `preset_location`, `format`
3. Separate the frontmatter from the structure content (everything after the closing `---`)

If the file has no frontmatter or is missing required fields:
```
Файл '<file>' не содержит метаданных outline.
Он был создан с помощью /outline? Без метаданных невозможно определить пресет.
```
Stop here.

### E-3: Load Preset and Agents

Based on `preset` and `preset_location` from metadata:

- `default` → load from `assets/default/` within the skill directory
- `local` → load from `.outline-presets/<preset>/`
- `global` → load from `~/.claude/outline-presets/<preset>/`

If the preset directory no longer exists (deleted since generation):
```
Пресет '<preset>' (<preset_location>) не найден.
Возможно, он был удалён после генерации этой структуры.
```
Stop here.

Load and validate pipeline (same as G-3). Use the `format` from metadata as the active format.

### E-4: Run Edit Pipeline

Initialize context variables:
```
topic         = topic from metadata
current_draft = structure content from file (without frontmatter)
feedback      = user's edit comment
iteration     = 1
output_format = <constructed from format in metadata, same as G-3>
```

**CRITICAL:** The generator receives the existing structure as `current_draft` and the user's edit comment as `feedback`. This triggers the generator's **re-run behavior** (feedback present) — it will address the edit comment while preserving the rest of the structure.

Run the pipeline cycle exactly as G-4:
1. Generator re-runs with `current_draft` + `feedback` → produces updated structure
2. Reviewer evaluates the updated structure
3. Loop until `APPROVED` or `max_iterations` reached

Then run post-loop agents (G-5) if any.

### E-5: Save and Report

Overwrite the **same file** with updated content:
- Preserve the original frontmatter (same `topic`, `preset`, `preset_location`, `format`)
- Replace the structure content with the new `current_draft`

Print:
```
Структура отредактирована.

  Файл:       <file path>
  Изменение:  <user's comment, truncated to ~60 chars>
  Итерации:   <number of iterations used> / <max_iterations>
```

---

## Review Procedure

### R-1: Parse Command

Extract from the user's input:
- **`<file>`** — path to a structure file (any markdown file with a presentation structure)
- **`--preset <name>`** — optional, specific preset to use for review

If the file is missing or not provided, ask: "Какой файл отревьюить? Укажите путь к файлу со структурой презентации."

### R-2: Read File

1. Read the file content
2. If the file has a YAML frontmatter (e.g., created by `/outline`), strip it — extract only the structure content
3. Infer the topic from the structure content (use the title or first heading)

If the file is empty or unreadable:
```
Не удалось прочитать файл '<file>'.
```
Stop here.

### R-3: Select Preset

**Path A: `--preset <name>` specified**

Look up the preset (local → global), same as G-2 Path A. Error if not found.

**Path B: Auto-select**

1. Scan for presets in both locations (same as G-2 Path B)
2. If presets exist, auto-select based on the inferred topic and structure content
3. If no presets exist or auto-selection returns "none" → use built-in default

**Path C: File has metadata frontmatter**

If the file was created by `/outline` and has `preset` + `preset_location` in frontmatter, prefer that preset (unless `--preset` flag overrides it).

### R-4: Load Preset and Pipeline

Load and validate the preset (same as G-3). Determine `output_format` from the preset's format field.

### R-5: Run Review Cycle

Initialize context variables:
```
topic         = inferred topic from R-2
current_draft = structure content from file
feedback      = "" (empty — no user comment, reviewer decides)
iteration     = 1
output_format = <constructed from preset format>
```

**CRITICAL:** Skip the initial generator step (`role: create`). The existing structure IS the first draft. Start directly with the reviewer.

Run the review→fix cycle from the pipeline:
1. Reviewer evaluates `current_draft`
2. If `APPROVED` → structure is good, report to user and stop (no changes needed)
3. If `NEEDS_REVISION`:
   - Fixer (or generator in no-fixer pattern) revises based on feedback → updates `current_draft`
   - Reviewer evaluates again
   - Loop until `APPROVED` or `max_iterations` reached

Then run post-loop agents (G-5) if any.

### R-6: Save and Report

If the structure was modified (reviewer did NOT approve on first pass):

Save the improved structure to a new file: `<original-filename>-reviewed.md`
- Add metadata frontmatter with `topic`, `preset`, `preset_location`, `format`
- Original file is NOT overwritten (user brought it, we don't touch it)

Print:
```
Ревью завершено.

  Исходный файл:  <original file>
  Пресет:         <preset name or "по умолчанию">
  Итерации:       <number of iterations used> / <max_iterations>
  Результат:      <file path of reviewed version>
```

If the structure was approved on first pass (no changes):
```
Ревью завершено — структура одобрена без изменений.

  Файл:    <original file>
  Пресет:  <preset name or "по умолчанию">
```

---

## Learn Procedure

### L-1: Parse Arguments

- If `preset` is specified:
  1. Look up by name: local (`.outline-presets/<name>/`) → global (`~/.claude/outline-presets/<name>/`)
  2. If not found, error:
     ```
     Пресет '<name>' не найден.
     Искали в: .outline-presets/<name>/, ~/.claude/outline-presets/<name>/
     ```
     Stop here.
  3. Train only this preset.

- If no preset specified:
  - Scan both local and global storage for all presets
  - If no presets found:
    ```
    Пресетов для обучения не найдено. Сначала создайте пресет:
      /outline --new-preset
    ```
    Stop here.
  - Train all found presets.

### L-2: Generate Test Topics and Run Pipelines

For each preset being trained:

1. Generate N test topics that match the preset's description and keywords. Topics should be diverse — vary industry, scope, and complexity.

2. For each test topic (i = 1 to N):
   a. Run the full Generate Procedure (G-1 through G-7) using this preset and topic
   b. Save the output to `learn-<preset-name>/run-<i>/outline.md`
   c. Save metadata (topic, iterations used, final verdict) to `learn-<preset-name>/run-<i>/meta.md`

### L-3: Critic Analysis

After all N runs complete, construct a critic agent on-the-fly. Dispatch it via the Agent tool with this prompt:

```
You are a demanding presentation structure critic. Analyze N generated
presentation outlines and evaluate the agent pipeline that produced them.

INPUTS:
- Preset metadata: <preset.md content>
- Pipeline: <pipeline.md content>
- Agent prompts: <all agents/*.md contents>
- Generated outlines: <all N outline outputs>
- Run metadata: <all N meta.md files — topics, iterations, verdicts>

ANALYSIS:
1. Evaluate each outline for: structure quality, logical flow, coverage,
   pacing, opening/closing strength
2. Look for SYSTEMIC patterns — issues that appear across multiple runs
3. Evaluate agent effectiveness:
   - Generator: Does it produce good first drafts?
   - Reviewer: Does it catch real issues? Is it too strict or too lenient?
   - Fixer (if any): Does it actually address feedback?
4. Check if the pipeline structure itself is effective:
   - Are there enough agents?
   - Is the loop count appropriate?
   - Would adding/removing agents help?

OUTPUT FORMAT:

# Critic Report: <preset-name>

## Overall Quality: X/10

## Systemic Issues

### Issue 1: [Title]
- **Severity**: critical | major | minor
- **Frequency**: N/N runs affected
- **Description**: What's wrong
- **Root cause**: Which agent's prompt allows this
- **Proposed fix**: Specific change to agent prompt — quote the
  relevant section, show before/after

## Pipeline-Level Suggestions
- [Any suggestions to modify pipeline.md — add/remove agents, change iterations]

## What Works Well
- [Patterns to preserve]
```

### L-4: Propose Changes

Based on the critic's report:

1. Generate a diff-style summary of all proposed changes:
   ```
   Предлагаемые изменения для пресета '<name>':

   1. [agent-name.md] — <brief description of change>
      - Было: "<relevant excerpt>"
      - Стало: "<proposed text>"

   2. [pipeline.md] — <brief description of change>
      - <details>

   Применить изменения? (да / нет / выбрать)
   ```

2. Wait for user confirmation:
   - **да** → apply all changes
   - **нет** → discard all changes
   - **выбрать** → let user pick which changes to apply (number them)

**CRITICAL:** Do NOT apply changes without user confirmation. Always show the diff summary and wait for explicit approval.

### L-5: Apply Changes

For each approved change:

1. Read the target file
2. Apply the modification using the Edit tool
3. Verify the edit was applied correctly

**Failure handling:**
- If an agent produced zero usable output across ALL N runs → mark as "critically broken", do NOT modify it, flag for user attention
- If partial runs failed → apply improvements from successful runs, note failures in report

### L-6: Report

Print a final summary:
```
Обучение завершено для пресета '<name>'.

  Прогоны:       N всего, M успешных
  Изменения:     X применено, Y отложено, Z пропущено (критически сломанные агенты)
  Изменённые файлы:
    - agents/<agent>.md — <what changed>
    - pipeline.md — <what changed> (if applicable)

  Агенты, требующие внимания:
    - <agent-name> — <reason>
```

Clean up: the `learn-<preset-name>/` directory with test outputs can be kept for reference or deleted — ask the user: "Удалить директорию с тестовыми результатами learn-<preset-name>/? (да/нет)"

---

## Deep Learn Procedure

Iterative training: run test pipelines, analyze, apply fixes to agent files, repeat N cycles. Each cycle builds on the improvements from the previous one.

**Key differences from `--learn=N`:**

| | `--learn=N` | `--deep_learn=N` |
|---|---|---|
| N means | number of test topics in one batch | number of training cycles |
| Test topics per cycle | N | 3 (fixed) |
| Agent file edits | 1 time at end, with user confirmation | after each cycle, automatically |
| Core idea | "test and suggest" | "train iteratively until convergence" |

### DL-1: Parse Arguments

- Parse N from the argument (e.g., `--deep_learn=5`). N must be >= 2 (minimum 2 cycles to see improvement). If N < 2, error:
  ```
  --deep_learn требует минимум 2 цикла (N >= 2).
  ```
  Stop here.

- If `preset` is specified:
  1. Look up by name: local (`.outline-presets/<name>/`) → global (`~/.claude/outline-presets/<name>/`)
  2. If not found, error:
     ```
     Пресет '<name>' не найден.
     Искали в: .outline-presets/<name>/, ~/.claude/outline-presets/<name>/
     ```
     Stop here.
  3. Train only this preset.

- If no preset specified:
  - Scan both local and global storage for all presets
  - If no presets found:
    ```
    Пресетов для обучения не найдено. Сначала создайте пресет:
      /outline --new-preset
    ```
    Stop here.
  - If multiple presets found, train each one independently (full DL-2 through DL-5 cycle for each).

### DL-2: Create Working Directory

Create the working directory for training artifacts:

```
deep-learn-<preset-name>/
  ├── cycle-1/
  │   ├── run-1/  (outline.md, meta.md)
  │   ├── run-2/
  │   ├── run-3/
  │   ├── critic-report.md
  │   └── changes-applied.md
  ├── cycle-2/
  │   └── ...
  ├── ...
  └── final-report.md
```

Also, **snapshot the original agent files** before any modifications:
- Copy all files from the preset's `agents/` directory to `deep-learn-<preset-name>/agents-snapshot-original/`
- This allows comparison of before/after at the end

Print:
```
Deep Learn: запуск итеративного обучения для пресета '<name>'

  Циклов:            N
  Тем за цикл:       3
  Всего прогонов:    N × 3
  Рабочая директория: deep-learn-<preset-name>/

  Цикл 1/<N> начинается...
```

### DL-3: Run Training Cycle

For each cycle `c` from 1 to N:

#### DL-3.1: Generate Test Topics

Generate 3 diverse test topics that match the preset's description and keywords. **CRITICAL:** Topics MUST differ between cycles — do not reuse topics from previous cycles. Vary:
- Industry / domain
- Scope (small pitch vs. large fund)
- Complexity (early-stage vs. mature studio)
- Geography

#### DL-3.2: Run Pipelines

For each test topic (i = 1 to 3):

1. **Re-read agent files from disk** before each cycle — agents may have been modified by the previous cycle's fixes
2. Run the full Generate Procedure (G-1 through G-7) using the current preset and topic
3. Save output to `deep-learn-<preset-name>/cycle-<c>/run-<i>/outline.md`
4. Save metadata (topic, iterations used, final verdict, which iteration got APPROVED) to `deep-learn-<preset-name>/cycle-<c>/run-<i>/meta.md`

**Run all 3 topics in parallel** using the Agent tool with `run_in_background: true` for maximum speed.

#### DL-3.3: Critic Analysis

After all 3 runs in this cycle complete, construct a critic agent (same prompt as L-3) with:
- Current agent prompts (read fresh from disk)
- All 3 generated outlines from this cycle
- All 3 meta.md files from this cycle
- If `c > 1`: also provide the previous cycle's critic report for context, so the critic can assess whether previous issues were fixed

Append this extra instruction to the critic prompt when `c > 1`:
```
IMPORTANT: This is cycle <c> of <N>. The previous cycle's critic report is
provided below. Evaluate whether the issues identified in the previous cycle
have been fixed. If an issue persists despite a fix attempt, escalate its
severity. If new issues appeared after fixes, flag them as REGRESSION.

Previous cycle critic report:
<previous critic report content>
```

Save the critic report to `deep-learn-<preset-name>/cycle-<c>/critic-report.md`.

#### DL-3.4: Auto-Apply Changes

**CRITICAL difference from `--learn`:** Changes are applied AUTOMATICALLY without user confirmation.

Based on the critic's report:

1. Extract all proposed fixes with severity `critical` or `major`
2. For `minor` severity: apply only if the fix is simple and low-risk (single-line change). Skip complex minor fixes.
3. For each change to apply:
   a. Read the target agent file from disk
   b. Apply the modification using the Edit tool
   c. Verify the edit was applied correctly
4. Save a log of applied changes to `deep-learn-<preset-name>/cycle-<c>/changes-applied.md`:
   ```markdown
   # Changes Applied — Cycle <c>

   ## Applied
   1. [file] — <description>
      - Severity: <level>
      - Before: "<excerpt>"
      - After: "<excerpt>"

   ## Skipped
   1. [file] — <description>
      - Reason: <why skipped>
   ```

**Safety rules for auto-apply:**
- NEVER delete an agent file or remove it from the pipeline
- NEVER change the pipeline structure (steps order, add/remove agents) — only modify agent prompts
- NEVER remove existing instructions from an agent — only add, refine, or rephrase
- If the critic suggests a pipeline-level change (add agent, change iterations), log it as a suggestion but do NOT auto-apply — save it for the final report
- If an edit fails (old_string not found), log it as skipped and continue

#### DL-3.5: Cycle Report

Print a brief status update:
```
Цикл <c>/<N> завершён.

  Качество:    <overall quality from critic>/10
  Прогоны:     3/3 успешных
  Итерации:    <avg iterations across 3 runs> (среднее)
  Правки:      <X applied, Y skipped>
  Регрессии:   <count, or "нет">

  Цикл <c+1>/<N> начинается...
```

If this is the last cycle (c = N), skip the "next cycle" line.

### DL-4: Convergence Detection (Optional Early Stop)

After DL-3.5, check for convergence:

- If the critic's overall quality score is **9 or 10** for two consecutive cycles → early stop:
  ```
  Раннее завершение: качество стабилизировалось на <score>/10 в циклах <c-1> и <c>.
  Дальнейшие итерации вряд ли дадут улучшение.
  ```
  Skip remaining cycles and proceed to DL-5.

- If quality **decreased** compared to the previous cycle (regression) → warn but continue:
  ```
  ⚠ Регрессия: качество упало с <prev>/10 до <curr>/10.
  Возможно, правки в цикле <c-1> ухудшили агентов. Продолжаем — следующий цикл попытается исправить.
  ```

### DL-5: Final Report

After all cycles complete (or early stop), generate a comprehensive final report.

Save to `deep-learn-<preset-name>/final-report.md` and print:

```
Deep Learn завершён для пресета '<name>'.

  Циклов выполнено: <actual cycles> / <N planned>
  Прогонов всего:   <total runs>

  Прогресс качества:
    Цикл 1: <score>/10
    Цикл 2: <score>/10
    ...
    Цикл N: <score>/10

  Всего правок применено: <total across all cycles>
  Регрессий обнаружено:   <count>

  Изменённые файлы (суммарно):
    - agents/<agent>.md — <cumulative summary of all changes>
    - ...

  Нереализованные предложения (требуют ручного вмешательства):
    - <pipeline-level suggestions that were not auto-applied>

  Рабочая директория: deep-learn-<preset-name>/
```

### DL-6: Snapshot Comparison

Read the original agent snapshots from `deep-learn-<preset-name>/agents-snapshot-original/` and compare with the current agent files on disk.

For each agent that changed, print a brief before/after summary:
```
Изменения агентов (до → после):

  <agent-name>.md:
    + <added instruction or refinement>
    ~ <rephrased section>
    ...
```

### DL-7: Cleanup

Ask the user:
```
Удалить рабочую директорию deep-learn-<preset-name>/? (да/нет)
Директория содержит: снимки агентов, все тестовые аутлайны, отчёты критиков, логи правок.
```

---

## Utility: Topic Slug Generation

Used by G-6 to create file names from topics.

**Rules:**
1. Convert to lowercase
2. Replace spaces with hyphens
3. Strip all characters that are not alphanumeric or hyphens
4. Collapse consecutive hyphens into a single hyphen
5. Remove leading/trailing hyphens
6. If longer than 50 characters, truncate at the last word boundary before 50 characters

**Examples:**
| Topic | Slug |
|-------|------|
| "Q4 Investor Pitch" | `q4-investor-pitch` |
| "Introduction to Machine Learning: A Primer" | `introduction-to-machine-learning-a-primer` |
| "Why Our Team Needs to Adopt CI/CD Practices ASAP!!!" | `why-our-team-needs-to-adopt-cicd-practices-asap` |
| "A Very Long Topic That Goes On And On And Definitely Exceeds Fifty Characters Limit" | `a-very-long-topic-that-goes-on-and-on-and` |
