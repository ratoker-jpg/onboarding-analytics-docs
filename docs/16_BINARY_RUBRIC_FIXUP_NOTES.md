# 16_BINARY_RUBRIC_FIXUP_NOTES.md

Дата фиксации: 23.06.2026

Этот файл фиксирует методологический fixup к PR с бинарными рубриками оценки собеседования и звонков.

## 1. Блок как единица оценки

Трёхбалльная шкала не используется.

```text
question = binary check
block = evaluation unit
stage = group of blocks
final score = weighted percent by blocks/stages
```

Отдельный вопрос не является итоговой оценкой кандидата или звонка. Вопрос нужен, чтобы собрать evidence и посчитать вклад в блок.

Для собеседования единица оценки — блок компетенции: motivation, responsibility, communication, learning_potential, antifragility, hard_sales_fit, segment_understanding, roleplay.

Для звонка единица оценки — этап продаж: contact, needs_discovery, presentation, objections, closing_next_step, correctness.

## 2. Интерпретация только через проценты

```text
block_score = 0–100%
stage_score = 0–100%
overall_score = 0–100%
```

Качественные зоны допустимы только как текстовые статусы:

```text
critical_gap
weak_area
working_minimum
strong
excellent
```

Баллы 1/2/3 не используются.

## 3. Mapping в candidate_scores

### Собеседование

```text
soft_score ← weighted result по soft competency blocks
hard_score ← hard_sales_fit + segment_understanding + roleplay sales structure
learning_score ← learning_potential + antifragility + feedback acceptance
risk_score ← red_flags / critical risks / conflict
```

### Звонки

```text
call_quality_score ← weighted average по звонкам start/middle/final
risk_score может повышаться при critical_errors, dangerous promises, грубых нарушениях мануала
```

## 4. objections_05

Вопрос `objections_05` про наличие возражения по релевантности продукта не должен участвовать в weighted score, потому что это не качество оператора, а характеристика реакции клиента.

Его нужно трактовать как metadata:

```text
relevance_objection_present: yes/no/not_enough_data
```

Веса блока возражений должны считаться без `objections_05`:

```text
objections_01 = 0.35
objections_02 = 0.25
objections_03 = 0.20
objections_04 = 0.20
```

## 5. Матрица зависимостей от источников

| Блок / вопросы | Нужный источник | Fallback, если источника нет |
|---|---|---|
| presentation_01–04 | product_dictionary | not_enough_data или conflict, не yes |
| close_03–04 | product_dictionary + client_card / target_action | not_enough_data |
| segment_understanding_* | segment_manual | not_checked |
| roleplay_* | roleplay_transcript + client_card / role | not_checked |
| motivation_* | interview_transcript + hr_notes | not_checked |
| hard_sales_fit_* | teamlead_notes + roleplay | not_checked |
| expert-dependent questions | expert_markup | not_enough_data |
| call stages | call_transcript | not_enough_data |

## 6. Evidence JSON

Во всех JSON-форматах результата должны быть поля:

```json
{
  "evidence": "",
  "quote": "",
  "source": "",
  "source_ref": ""
}
```

`quote` — короткая цитата из транскрибации или заметок.  
`source_ref` — строка, таймкод или ID фрагмента, если доступно.  
Если `source_ref` нет, вернуть пустую строку.  
`source_ref` нельзя придумывать.

## 7. Что не меняем

Не добавляем код, миграции, server endpoints, AI-вызовы, dashboard, HTML report generation.
