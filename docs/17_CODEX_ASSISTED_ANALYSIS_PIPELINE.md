# Phase 3E0 — Codex-assisted Analysis Pipeline

## Статус

Документ фиксирует целевую схему следующего технического этапа перед постановкой задачи в GLM/Codex.

Этот этап нужен не для подключения платного AI API, а для безопасного полуавтоматического анализа через Codex/LLM-исполнителя по подготовленным данным и строгому JSON-контракту.

## Почему нужен этот этап

Сейчас система уже умеет:

- находить новичка по `base_key`;
- хранить часть данных по кандидату;
- показывать rich report;
- хранить итоговые `candidate_scores`;
- держать машинные рубрики в JSON;
- считать score через scoring engine.

Но пока отсутствует слой:

```text
raw transcript / notes / calls
        ↓
analysis result by rubric
        ↓
candidate_scores + выводы + evidence
```

То есть загруженные данные ещё не превращаются автоматически в оценку по рубрике.

## Целевая идея

Пользователь не должен вручную собирать HTML-отчёт или писать итоговую аналитику.

Целевой процесс:

```text
admin uploads raw data
        ↓
server stores raw data
        ↓
Codex exports candidate bundle
        ↓
Codex analyzes bundle by prepared prompt
        ↓
Codex returns strict JSON result
        ↓
server validates JSON
        ↓
server scoring engine calculates scores
        ↓
server saves analysis_runs and candidate_scores
        ↓
report-v1.html shows result
```

Codex в этой схеме — внешний аналитик, а не прямой редактор базы.

## Ключевой принцип безопасности

Codex не должен писать в SQLite напрямую.

Запрещённый подход:

```text
Codex opens DB → writes SQL manually → report somehow changes
```

Правильный подход:

```text
export bundle → analyze → strict JSON → import script → validation → scoring → save
```

## Роли компонентов

### Server

Сервер отвечает за:

- хранение raw data;
- экспорт безопасного bundle;
- загрузку rubric config;
- валидацию analysis result;
- расчёт score через scoring engine;
- запись `analysis_runs`;
- обновление `candidate_scores`;
- отдачу результата в `report-v1.html`.

### Codex / LLM executor

Codex отвечает только за:

- чтение exported bundle;
- применение подготовленного prompt;
- заполнение `question_results` по рубрике;
- evidence, quote, source, source_ref;
- summary, strengths, growth_zones, red_flags, coach_recommendations;
- возврат строгого JSON.

Codex не должен:

- менять код без отдельной задачи;
- писать SQL;
- придумывать evidence;
- использовать данные вне bundle;
- возвращать свободный текст вместо JSON.

### Report

`report-v1.html` не анализирует данные. Он только показывает:

- candidate info;
- completeness;
- scores;
- call_stats;
- interview_summary;
- ops_summary;
- training_bot_dialogs;
- analysis results после импорта.

## Что должно появиться в Phase 3E0

### 1. Export candidate analysis bundle

Скрипт:

```bash
node scripts/export_candidate_analysis_bundle.js --base-key GUF97MUL --type interview --out tmp/GUF97MUL_interview_bundle.json
node scripts/export_candidate_analysis_bundle.js --base-key GUF97MUL --type calls --out tmp/GUF97MUL_calls_bundle.json
```

Bundle должен включать:

```json
{
  "base_key": "GUF97MUL",
  "analysis_type": "interview",
  "candidate": {},
  "completeness": {},
  "scores": {},
  "manual_inputs": [],
  "training_bot_dialogs": [],
  "call_stats": {},
  "ops_summary": {},
  "interview_summary": {},
  "rubric": {},
  "source_refs": []
}
```

Для `interview` подключается:

```text
config/rubrics/interview_binary_v1.json
```

Для `calls` подключается:

```text
config/rubrics/calls_automanual_binary_v1.json
```

### 2. Prompt templates

Нужны файлы:

```text
prompts/codex/interview_analysis_v1.md
prompts/codex/calls_analysis_v1.md
```

Prompt должен требовать:

- анализировать только bundle;
- не выдумывать факты;
- каждый `yes` / `no` подтверждать evidence;
- если данных нет — ставить `not_checked` / `not_enough_data`;
- возвращать только JSON;
- не писать в БД;
- не менять код;
- после подготовки JSON запускать import script отдельно.

### 3. Analysis result schema / validator

Нужен строгий контракт результата:

```json
{
  "schema_version": "analysis_result_v1",
  "base_key": "GUF97MUL",
  "analysis_type": "interview",
  "rubric_id": "interview_binary_v1",
  "rubric_version": "1.0.0",
  "question_results": [
    {
      "question_id": "soft_responsibility_01",
      "answer": "yes",
      "evidence": "Кандидат описал конкретный пример...",
      "quote": "короткая цитата",
      "source": "interview_transcript",
      "source_ref": "line:120"
    }
  ],
  "summary": "",
  "strengths": [],
  "growth_zones": [],
  "red_flags": [],
  "coach_recommendations": [],
  "risk_flags": []
}
```

Validator должен проверять:

- `base_key` есть;
- `analysis_type` допустимый;
- `rubric_id` соответствует типу анализа;
- `question_id` существует в rubric;
- `answer` входит в allowed answers;
- для `yes` требуется evidence;
- неизвестные dangerous fields не принимаются;
- результат не содержит secrets.

### 4. Import analysis result script

Скрипт:

```bash
node scripts/import_analysis_result.js --file tmp/GUF97MUL_interview_result.json
```

Скрипт должен:

1. Прочитать JSON.
2. Валидировать результат.
3. Загрузить нужную rubric.
4. Посчитать score через `services/phase1_rubric_score_service.js`.
5. Сохранить запись в `analysis_runs`.
6. Обновить `candidate_scores` partial fields.
7. Не перетирать вручную заполненные поля без явного правила.

### 5. Dry-run mode

Для безопасной проверки:

```bash
node scripts/import_analysis_result.js --file tmp/GUF97MUL_interview_result.json --dry-run
```

Dry-run должен показать:

- validation result;
- calculated rubric score;
- candidate_scores patch;
- какие записи будут созданы/обновлены;
- без записи в БД.

### 6. Example files

Добавить примеры без персональных данных:

```text
examples/analysis/interview_result_example.json
examples/analysis/calls_result_example.json
```

### 7. Usage doc

Добавить инструкцию:

```text
docs/CODEX_ASSISTED_ANALYSIS_PIPELINE.md
```

Workflow:

```text
1. Upload data in admin.
2. Export bundle.
3. Give bundle + prompt to Codex.
4. Codex returns strict JSON.
5. Run dry-run import.
6. Run import.
7. Open report-v1.html.
```

## Как это будет выглядеть на Зорине Иване

Пример кандидата:

```text
base_key: GUF97MUL
candidate: Зорин Иван
```

Ожидаемый процесс:

1. Пользователь выбирает кандидата `GUF97MUL`.
2. Загружает транскрибацию собеседования, HR notes, teamlead notes, звонки, метрики и комментарии.
3. Codex экспортирует bundle:

```bash
node scripts/export_candidate_analysis_bundle.js --base-key GUF97MUL --type interview --out tmp/GUF97MUL_interview_bundle.json
```

4. Codex анализирует bundle по `prompts/codex/interview_analysis_v1.md`.
5. Codex сохраняет результат в JSON:

```text
tmp/GUF97MUL_interview_result.json
```

6. Запускается dry-run:

```bash
node scripts/import_analysis_result.js --file tmp/GUF97MUL_interview_result.json --dry-run
```

7. После проверки запускается import:

```bash
node scripts/import_analysis_result.js --file tmp/GUF97MUL_interview_result.json
```

8. `report-v1.html?base_key=GUF97MUL` показывает обновлённую аналитику.

## Что не входит в Phase 3E0

Не добавлять:

- OpenAI / external AI API calls;
- автоматические фоновые AI jobs;
- новые external dependencies;
- PDF;
- report snapshots;
- dashboard redesign;
- admin redesign;
- прямые SQL-инструкции для Codex;
- запись в БД без validator/scoring layer.

## Риски

### Риск 1. Codex выдумает evidence

Контроль:

- prompt запрещает выдумывать;
- обязательны `quote`, `source`, `source_ref`;
- если evidence нет — `not_checked` / `not_enough_data`.

### Риск 2. Codex вернёт красивый текст вместо JSON

Контроль:

- strict JSON schema;
- import script принимает только валидный JSON;
- отчёт не обновляется без успешного import.

### Риск 3. Случайная перезапись ручных оценок

Контроль:

- import script должен считать patch;
- dry-run обязателен перед import;
- не перетирать manual fields без явного режима.

### Риск 4. Утечка секретов

Контроль:

- export bundle не включает env;
- validator проверяет forbidden secret-like fields;
- scripts не выводят ADMIN_KEY / VIEWER_KEY.

## Критерий готовности Phase 3E0

Этап готов, если можно выполнить цепочку:

```text
export bundle → analyze by prompt → import JSON dry-run → import JSON → report updates
```

без:

- AI API;
- прямого SQL;
- ручного редактирования `candidate_scores`;
- ручной сборки HTML-отчёта.
