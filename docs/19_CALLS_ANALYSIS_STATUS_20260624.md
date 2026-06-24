# Phase 3E — статус механики анализа реальных звонков

Дата фиксации: 24.06.2026  
Проект кода: `ratoker-jpg/onboarding-cross-node`  
Документ фиксирует уже принятые решения и фактическую цепочку анализа звонков после PR #11–#17.

---

## 1. Что уже сделано

### 1.1. Разделены реальные звонки и учебные агенты

Для `analysis_type = calls` используются только реальные звонки новичка.

Разрешённые источники:

```text
manual_inputs.calls_start
manual_inputs.calls_middle
manual_inputs.calls_final
candidate_files.calls_start
candidate_files.calls_middle
candidate_files.calls_final
```

Запрещённые источники для анализа звонков:

```text
training_bot_dialogs
training_bot
bot_training
ROLE-*
dialog:*
result_payload
учебные агенты
```

Учебные агенты остаются отдельным типом данных и не должны попадать во вкладку «Звонки» как реальные звонки.

---

### 1.2. Файлы транскриптов нарезаются на отдельные звонки

Раньше система считала:

```text
1 transcript-файл = 1 звонок
```

Это было неверно: один файл может содержать несколько звонков подряд.

Теперь `scripts/export_candidate_analysis_bundle.js` для `analysis_type = calls` формирует:

```text
real_calls[]
```

где каждый объект — отдельный реальный звонок.

Формат:

```json
{
  "stage": "start",
  "stage_label": "Начало",
  "call_index": 1,
  "source_type": "candidate_file",
  "source_ref": "candidate_files.calls_start:6#call_1",
  "file_id": 6,
  "original_name": "_.txt",
  "transcript": "..."
}
```

Этапы:

```text
start  = Начало
middle = Середина
final  = Выпуск
```

Smoke по `GTRAIN02` подтвердил:

```text
real_calls: 9
by_stage: { start: 3, middle: 3, final: 3 }
```

---

### 1.3. Обновлён calls automanual rubric до v1.1.0

Актуальная рубрика:

```text
calls_automanual_binary_v1 v1.1.0
```

В рубрике 16 вопросов:

```text
contact:      2
needs:        4
presentation: 4
objections:   4
close:        2
```

Все веса внутри каждого этапа суммируются до `1.0`.

#### contact / Установление контакта

| question_id | source_code | Вопрос | Вес |
|---|---|---|---:|
| `contact_bank_tochka` | В4 | Сообщил ли оператор, что звонит из Банка Точка? | 0.2 |
| `contact_call_reason` | В5 | Сообщил ли оператор причину звонка? | 0.8 |

#### needs / Выявление потребности

| question_id | source_code | Вопрос | Вес |
|---|---|---|---:|
| `needs_business_activity` | В6 | Спросил ли оператор, каким бизнесом занимается клиент? | 0.1 |
| `needs_business_problems` | В7 | Спросил ли оператор клиента, с какими проблемами и трудностями он сталкивается в бизнесе? | 0.4 |
| `needs_problem_frequency` | В8 | Уточнил ли оператор, как часто клиент сталкивается с проблемами и трудностями в бизнесе? | 0.1 |
| `needs_current_solution` | В9 | Уточнил ли оператор у клиента, как он решает проблемы и трудности в бизнесе? | 0.4 |

#### presentation / Презентация

| question_id | source_code | Вопрос | Вес |
|---|---|---|---:|
| `presentation_any_product` | В11 | Презентовал ли оператор какие-либо продукты банка? | 0.35 |
| `presentation_asked_opinion` | В12 | Спрашивал ли оператор мнение клиента хотя бы об одном представленном продукте банка? | 0.1 |
| `presentation_calculation` | В13 | Подкрепил ли оператор выгоду от презентованных продуктов банка конкретными расчётами? | 0.2 |
| `presentation_linked_to_problem` | В14 | Презентовал ли оператор продукты банка, сославшись на озвученную клиентом проблему? | 0.35 |

#### objections / Отработка возражений

| question_id | source_code | Вопрос | Вес |
|---|---|---|---:|
| `objections_attempted_handle` | В15 | Пытался ли оператор снять возражения клиента по представленным продуктам банка? | 0.4 |
| `objections_checked_remaining_doubts` | В16 | Уточнил ли оператор, остались ли у клиента сомнения после ответа на его возражения по представленным продуктам банка? | 0.2 |
| `objections_used_examples` | В17 | Приводил ли оператор примеры того, как он сам или клиенты банка пользовались представленным продуктом? | 0.15 |
| `objections_compared_competitors` | В18 | Сравнивал ли оператор предложенные продукты или условия по ним с предложениями конкурентов? | 0.25 |

#### close / Тест на сделку

| question_id | source_code | Вопрос | Вес |
|---|---|---|---:|
| `close_purchase_action` | В20 | Предложил ли оператор клиенту совершить действия, необходимые для приобретения продукта? | 0.6 |
| `close_specific_next_step` | В21 | Договорился ли оператор с клиентом о конкретном следующем шаге с указанием даты или срока? | 0.4 |

Видимые B-коды не должны попадать в текст вопроса; они хранятся только в `source_code`.

---

### 1.4. Добавлен словарь продуктов Точки

Проблема: вопросы `presentation_*` и `objections_*` невозможно честно оценить без справочника продуктов. Иначе любое упоминание «сервиса» может ошибочно засчитаться как продукт банка.

Теперь для `analysis_type = calls` bundle содержит:

```text
product_dictionary[]
```

Источник:

```text
Google Sheet: 1grwKJPJ3VH6OE0Ky5v3J4FZVADHm7tJrFPwoppkdakI
Лист: Лист1
```

Колонки:

```text
A: Продукт Точки
B: Продукт ЧП?
C: Сегмент в который продается
D: Тип продукта
E: Описание продукта
F: Сокращения и аббревиатуры
G: Статус
H: Субкруг, который продает
```

Формат объекта:

```json
{
  "product_id": "platnaya_onlain_buhgalteriya",
  "product_name": "Платная онлайн-бухгалтерия",
  "raw_product_name": "Платная онлайн-бухгалтерия Отредактировано Калинкина",
  "is_chp": true,
  "segments": "...",
  "product_type": "...",
  "description": "...",
  "aliases": ["ОБ", "ПОБ", "Платная ОБ"],
  "status": "Продается",
  "selling_circle": "...",
  "source_ref": "products_sheet:row_18"
}
```

Правила фильтрации:

- исключать только явные статусы `Не продается`, `Не продаётся`, `Не продают`;
- также исключать строки, где описание прямо говорит, что кроссы не продают продукт;
- остальные статусы включать, включая совместные акции и самоходное подключение;
- если Google Sheet недоступен или после фильтрации словарь пустой, calls export должен падать, а не писать bundle.

Smoke по `GTRAIN02` подтвердил:

```text
product_dictionary: 39
excluded_status_leaks: 0
dirty_product_names: 0
```

---

### 1.5. Очистка названий продуктов

В Google Sheet могут попадать служебные пометки:

```text
Отредактировано Петрова
Отредактировано Калинкина
```

Они не должны попадать в `product_name`.

Текущая логика:

- `raw_product_name` сохраняет исходную ячейку;
- `product_name` очищается от `Отредактировано ...`;
- whitespace нормализуется;
- если после очистки имя пустое, строка пропускается.

Пример:

```json
{
  "product_name": "Платная онлайн-бухгалтерия",
  "raw_product_name": "Платная онлайн-бухгалтерия Отредактировано Калинкина"
}
```

---

### 1.6. Codex-result должен быть согласован с детализацией

Dry-run валидирует форму JSON, но не всегда ловит смысловые расхождения.

Был найден конфликт:

```text
call_quality_score: 88
avg(call_results): 78.33
```

Решение: результат был исправлен так, чтобы итоговая оценка считалась из `call_results`.

Актуальное состояние результата для `GTRAIN02`:

```text
call_quality_score: 78.3
avg(call_results): 78.33
question_results: 16
validation: PASS
```

Правило на будущее:

```text
Итоговая call_quality_score должна быть математически согласована с call_results и stage_dynamics.
```

---

## 2. Как должна работать финальная цепочка

```text
1. Загрузить реальные звонки в calls_start / calls_middle / calls_final.
2. Export bundle для analysis_type=calls.
3. Export нарезает transcript-файлы на real_calls[].
4. Export подтягивает product_dictionary из Google Sheets.
5. Codex анализирует только real_calls[] + rubric + product_dictionary[].
6. Codex возвращает question_results, stage_dynamics, call_results, products_detected.
7. import_analysis_result --dry-run должен быть PASS.
8. Проверить математическую согласованность score с call_results.
9. Импортировать результат.
10. Viewer/report показывает качество звонков, динамику и продукты.
```

---

## 3. Формат обязательного результата calls analysis

### 3.1. `question_results`

Должно быть 16 объектов по актуальным `question_id` из v1.1.0.

### 3.2. `stage_dynamics`

```json
{
  "start": {
    "label": "Начало",
    "overall": 4.06,
    "percent": 81.25,
    "blocks": {
      "contact": 1,
      "needs": 0.75,
      "presentation": 1,
      "objections": 0.42,
      "close": 0.83
    },
    "comment": "..."
  }
}
```

`overall` — шкала `0–5`.  
`percent` — шкала `0–100`.  
`blocks` — шкала `0–1`.

### 3.3. `call_results`

Должен быть один объект на каждый звонок из `real_calls[]`.

```json
{
  "stage": "start",
  "call_index": 1,
  "source_ref": "candidate_files.calls_start:6#call_1",
  "overall_percent": 73,
  "blocks": {
    "contact": 1,
    "needs": 0.75,
    "presentation": 0.8,
    "objections": 0.4,
    "close": 0.6
  },
  "products_detected": [
    {
      "product_id": "platnaya_onlain_buhgalteriya",
      "product_name": "Платная онлайн-бухгалтерия",
      "matched_by": "alias",
      "matched_text": "ОБ",
      "source_ref": "candidate_files.calls_start:6#call_1"
    }
  ],
  "comment": "..."
}
```

---

## 4. Product-dependent questions

Эти вопросы требуют `product_dictionary`:

```text
presentation_any_product
presentation_asked_opinion
presentation_linked_to_problem
objections_attempted_handle
objections_checked_remaining_doubts
objections_used_examples
objections_compared_competitors
```

Правила:

- продукт засчитывается только если найден в `product_dictionary`;
- поиск возможен по `product_name`, `aliases`, `description`;
- продукт, упомянутый только клиентом, не засчитывается как презентация оператора;
- общие фразы «у нас есть сервисы» не засчитываются;
- возражение засчитывается только если относится к найденному продукту;
- сравнение с конкурентом засчитывается только если оператор сравнил условия/продукт Точки с конкретным конкурентным продуктом/условием.

---

## 5. Нормы времени на трубке

Согласованные нормы:

| Период | Красная зона | Жёлтая зона | Зелёная зона |
|---|---:|---:|---:|
| День 3 / первые звонки | `<30 мин` | `30–44 мин` | `45+ мин` |
| День 4 / середина | `<60 мин` | `60–74 мин` | `75+ мин` |
| День 5 / выпуск | `<90 мин` | `90–104 мин` | `105+ мин` |
| Итого | `<170 мин` | `170–199 мин` | `200+ мин` |

Плейсхолдер «Шкалы качества по времени на трубке будут добавлены после согласования норм» больше не актуален.

---

## 6. Known next step

Если ещё не выполнено в рабочем окружении:

```bash
node scripts/import_analysis_result.js \
  --file tmp/GTRAIN02_calls_result_FINAL_PRODUCTS.json
```

После импорта:

1. перезапустить сервер;
2. открыть `report-v1.html?base_key=GTRAIN02`;
3. проверить вкладку «Звонки»:
   - score около `78.3`;
   - `stage_dynamics` начало/середина/выпуск;
   - 9 отдельных звонков;
   - продукты в разборе звонков;
   - слабая зона: отработка возражений.
