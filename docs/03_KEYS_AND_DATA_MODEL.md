# Ключи и модель данных

## Главная проблема

Старый подход:

```text
1 кандидат = 1 ключ = 1 строка сборки = 1 набор результатов
```

Для новой системы это не подходит.

Причина: у одного новичка будет много этапов, курсов, звонков, попыток и результатов.

Если всё хранить в одной строке, данные будут:

- перетираться;
- слипаться;
- плохо сравниваться;
- плохо масштабироваться;
- зависеть от формул Google Sheets.

## Новый принцип

```text
base_key = человек
session_key = конкретное действие / этап / звонок
```

## Base key

Базовый ключ создаётся один раз в начале.

Пример:

```text
MDLK5Y
```

Это ключ новичка.

Он используется для:

- входа новичка;
- связки всех данных;
- поиска карточки;
- группировки результатов;
- построения дашборда.

## Session key

Дочерний ключ создаётся под конкретный этап.

Формат:

```text
BASEKEY-STAGE-ATTEMPT
```

Примеры:

```text
MDLK5Y-TD-01
MDLK5Y-OB-01
MDLK5Y-OB-02
MDLK5Y-EDO-01
MDLK5Y-FUNDS-01
MDLK5Y-REAL-S-01
MDLK5Y-REAL-M-01
MDLK5Y-REAL-F-01
```

Где:

| Часть | Значение |
|---|---|
| `MDLK5Y` | базовый ключ новичка |
| `OB` | этап / продукт / курс |
| `01` | попытка / номер сессии |

## Правило

Нельзя заменять основной ключ кандидата на session_key.

Правильно:

```text
Кандидат.Ключ = MDLK5Y
Учебный звонок = MDLK5Y-OB-01
```

Неправильно:

```text
Кандидат.Ключ = MDLK5Y-OB-01
```

Это сломает вход, связки и старые данные.

## Коды этапов

Рекомендуемые stage codes:

| Код | Значение |
|---|---|
| `INT` | собеседование |
| `TD` | тестовый день |
| `OB` | онлайн-бухгалтерия |
| `EDO` | ЭДО |
| `FUNDS` | фонды |
| `CALL1` | учебный звонок 1 |
| `CALL2` | учебный звонок 2 |
| `REAL-S` | реальные звонки: старт |
| `REAL-M` | реальные звонки: середина |
| `REAL-F` | реальные звонки: выпуск |
| `FINALTEST` | итоговый тест |

## Минимальная серверная модель

### `candidates`

Хранит человека.

Поля:

- `id`;
- `base_key`;
- `full_name`;
- `last_name`;
- `first_name`;
- `segment`;
- `direction`;
- `status`;
- `mentor`;
- `recruiter`;
- `test_day_date`;
- `onboarding_start_date`;
- `created_at`;
- `updated_at`.

### `stage_keys`

Хранит выданные session_key.

Поля:

- `id`;
- `candidate_id`;
- `base_key`;
- `session_key`;
- `stage_code`;
- `stage_name`;
- `attempt_no`;
- `status`;
- `issued_at`;
- `used_at`;
- `expires_at`;
- `source`.

### `course_progress`

Хранит прохождение маршрута новичка.

Поля:

- `id`;
- `candidate_id`;
- `base_key`;
- `course_code`;
- `day_id`;
- `block_id`;
- `material_id`;
- `status`;
- `opened_at`;
- `completed_at`;
- `duration_sec`;
- `payload_json`.

### `training_bot_dialogs`

Хранит учебные диалоги из бота учебки.

Поля:

- `id`;
- `candidate_id`;
- `base_key`;
- `session_key`;
- `role_id`;
- `dialog_date`;
- `transcript_text`;
- `source_sheet`;
- `raw_payload_json`.

### `training_bot_roles`

Хранит портреты ролей.

Поля:

- `role_id`;
- `team_id`;
- `company_name`;
- `client_name`;
- `position`;
- `tax_system`;
- `business_type`;
- `previous_interactions`;
- `client_info`;
- `business_experience`;
- `organization_form`;
- `success_criteria`;
- `failure_criteria`;
- `target_action`;
- `objections`;
- `tone`;
- `additional_info`.

### `voice_bot_sessions`

Хранит результаты голосовых агентов.

Поля:

- `id`;
- `candidate_id`;
- `base_key`;
- `session_key`;
- `stage_code`;
- `provider`;
- `external_session_id`;
- `status`;
- `score`;
- `started_at`;
- `finished_at`;
- `raw_payload_json`.

### `real_calls`

Хранит реальные звонки новичка.

Поля:

- `id`;
- `candidate_id`;
- `base_key`;
- `period` — `start`, `middle`, `final`;
- `call_date`;
- `transcript_text`;
- `audio_url`;
- `manual_score_json`;
- `ai_analysis_json`.

### `operational_metrics`

Хранит ручные/операционные показатели.

Поля:

- `id`;
- `candidate_id`;
- `base_key`;
- `date`;
- `training_day`;
- `talk_time_minutes`;
- `calls_count`;
- `calls_over_2_min_percent`;
- `goals_filled_status`;
- `statuses_filled_status`;
- `overdue_goals_count`;
- `comment`.

### `final_tests`

Хранит итоговый тест.

Поля:

- `id`;
- `candidate_id`;
- `base_key`;
- `test_name`;
- `score`;
- `max_score`;
- `errors_json`;
- `weak_topics_json`;
- `report_url`;
- `raw_payload_json`.

### `report_snapshots`

Хранит результаты AI-анализа и HTML/JSON отчёты.

Поля:

- `id`;
- `candidate_id`;
- `base_key`;
- `snapshot_type`;
- `generated_at`;
- `json_payload`;
- `html`;
- `markdown`;
- `xlsx_export_url`.

## Отношения

```text
candidates.base_key
  ├─ stage_keys.base_key
  ├─ course_progress.base_key
  ├─ training_bot_dialogs.base_key
  ├─ voice_bot_sessions.base_key
  ├─ real_calls.base_key
  ├─ operational_metrics.base_key
  ├─ final_tests.base_key
  └─ report_snapshots.base_key
```

## Рекомендация по хранилищу

Для MVP:

```text
SQLite
```

Причины:

- быстрее внедрить;
- достаточно для одного сервера;
- можно делать backup одним файлом;
- проще, чем Postgres;
- лучше Google Sheets как источник истины.

Для более серьёзной версии:

```text
Postgres
```

Если будут:

- несколько процессов;
- много пользователей;
- webhooks;
- права доступа;
- стабильная эксплуатация.
