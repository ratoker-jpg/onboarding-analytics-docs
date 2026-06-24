# Rich Report v5 — implementation roadmap

**Дата:** 2026-06-24  
**Проект:** аналитика новичков для кросс-продаж  
**Код:** `ratoker-jpg/onboarding-cross-node`  
**Документация:** `ratoker-jpg/onboarding-analytics-docs`  
**Текущий тестовый кандидат:** `GTRAIN02` — Иванов Иван, `Грузоперевозки`, направление `S`  
**Целевой viewer:** `/onboarding_cross/report-v1.html?base_key=GTRAIN02`

---

## 1. Зачем этот документ

Этот документ фиксирует контекст после трёх входов:

1. Целевой HTML-прототип `ivanov_ivan_report_final_v5.html`.
2. Отчёт по правкам `ivanov_ivan_report_final_v5_changes.md`.
3. Read-only аудит Opus `AUDIT_ONBOARDING_ANALYTICS_2026_06_24.md`.

Цель — не потерять продуктовый образ результата и разложить внедрение на маленькие проверяемые PR.

---

## 2. Главный вывод

Целевой формат отчёта уже понятен: **v5 HTML — это эталон UI/UX для карточки новичка**.

Но внедрять его надо не копированием статического HTML целиком, а через аккуратную интеграцию в текущий `public/report-v1.html` и viewer payload.

Главный риск из аудита Opus остаётся выше визуальных правок: **импорт результата звонков сейчас должен быть защищён семантической сверкой**. Нельзя продолжать масштабировать отчёт, если в БД может попасть несогласованный анализ звонков.

---

## 3. Целевой образ отчёта v5

### 3.1. Общая структура

Целевой отчёт сохраняет 7 вкладок:

1. Основной отчёт.
2. Оценка по этапам.
3. Звонки.
4. Собеседование.
5. Учебные агенты.
6. Операционка.
7. Рекомендации.

Это текущий целевой scope. Не расширять до 10 вкладок из старой спецификации, пока не стабилизирован v5.

### 3.2. Основной отчёт

Нужно оставить:

- ФИО: `Иванов Иван` для демо.
- Сегмент.
- Направление.
- Даты этапов без времени.
- Опыт кандидата в верхней части.
- Краткое резюме для тренера.
- Блок звонков в формате **общего анализа по автомануалу**.
- Ключевой вывод.
- Сильные стороны.
- Зоны роста.
- Риски.
- Рекомендации тренеру.

Нужно убрать / не возвращать:

- ключ новичка в пользовательском HTML;
- лишнюю «Сводку цифр»;
- субъективные блоки: итоговый статус, риск, готовность данных, отдельный следующий шаг для тимлида;
- дублирование динамики «Начало / Середина / Выпуск» в основном отчёте.

### 3.3. Блок «Звонки» в основном отчёте

В основном отчёте больше не показывать три карточки `Начало / Середина / Выпуск`.

Вместо этого показывать общий блок:

- Установление контакта.
- Выявление потребности.
- Презентация продукта.
- Работа с возражениями.
- Тест на сделку и завершение звонка.
- Общий вывод анализа.

Динамика `Начало / Середина / Выпуск` остаётся только во вкладке **«Звонки»**.

### 3.4. Вкладка «Звонки»

Оставить метрики:

- время на трубке;
- всего звонков;
- доля звонков больше 2 минут.

Убрать / не возвращать:

- дозвоны;
- звонки больше 10 минут;
- эффективные минуты;
- технические подписи вроде `% от ориентира` и `Ориентир при средней 1:17`.

Добавить / сохранить:

- цветовые шкалы по времени на трубке;
- цветовые шкалы по дням;
- управленческую формулировку для количества звонков:  
  **«Количество звонков ниже ожидаемого при таком объёме времени.»**

### 3.5. Логика зоны «Всего звонков»

Для текущего примера:

- факт времени на трубке: `214 минут`;
- ориентир по количеству звонков: `180 звонков`;
- факт звонков: `89 звонков`;
- выполнение: `89 / 180 = 49,4%`.

Зоны:

| Зона | Диапазон |
|---|---|
| Зелёная | 80–125% от ориентира |
| Жёлтая | 45–79% или 126–145% от ориентира |
| Красная | ниже 45% или выше 145% от ориентира |

Почему `89` — жёлтая зона:

- новичок набрал хороший объём минут;
- количество звонков примерно в 2 раза ниже ожидаемого;
- это не провал активности, но есть риск слишком длинных разговоров без достаточного количества попыток.

### 3.6. Собеседование / учебные агенты / операционка

Собеседование:

- оставить прежнюю структуру;
- не показывать текстовый фрагмент собеседования в основном интерфейсе.

Учебные агенты:

- оставить вкладку;
- блоки оценки по умолчанию свернуть;
- показывать как учебные диалоги, не как реальные звонки;
- не оценивать их рубрикой реальных звонков.

Операционка:

- пока оставить без структурного пересбора.

### 3.7. Рекомендации

Оставить блоки:

- что учитывается;
- что пока не учтено;
- итоговая рекомендация;
- сильные стороны;
- зоны роста;
- риски;
- рекомендации тренеру;
- что контролировать в первые 1–2 недели.

Формулировка:

- `Красные флаги` заменить на `Риски`.
- Английский текст не должен попадать в пользовательский UI.

---

## 4. Главные выводы из аудита Opus

### 4.1. Что работает

- Слоистая архитектура проекта уже нормальная.
- Есть viewer API, admin API, SQLite-слой, import pipeline.
- Разделение реальных звонков и учебных агентов в целом реализовано правильно.
- Rubric scoring работает.
- `calls_automanual_binary_v1 v1.1.0` содержит 16 вопросов.
- `product_dictionary` подтягивается и чистится.
- `report-v1.html` уже частично поддерживает rich report.

### 4.2. Главный P0 риск

Импорт результата звонков не должен полагаться только на формальную JSON-валидацию.

Нужна семантическая сверка:

- `question_results.length == 16`;
- `call_results.length == real_calls.length`;
- `stage_dynamics` содержит `start / middle / final`;
- средняя по `call_results` согласована с итоговым score из rubric engine;
- запрещённые источники учебных агентов не попали в `analysis_type=calls`.

Если сверка не проходит — live import должен останавливаться.

### 4.3. Что не делать сейчас

Не делать:

- большой rewrite;
- смену стека;
- перенос на другой фреймворк;
- расширение dashboard до сложной BI-системы;
- интеграцию новых вкладок, пока v5 не стабилен;
- Codex/Opus-анализ без `real_calls[] + rubric v1.1.0 + product_dictionary[]`;
- импорт результата, если score не согласован с детализацией.

---

## 5. Implementation roadmap

## Phase 0 — зафиксировать актуальное состояние

**Цель:** убедиться, что GTRAIN02 импортирован и viewer показывает актуальную версию.

### Задачи

| ID | Задача | Репозиторий | Файлы | Риск | Проверка |
|---|---|---|---|---|---|
| P0.1 | Импортировать `tmp/GTRAIN02_calls_result_FINAL_PRODUCTS.json`, если ещё не импортирован | onboarding-cross-node | DB через script | Средний | dry-run PASS, live import PASS |
| P0.2 | Перезапустить сервер | server | runtime | Низкий | `tail app.log`, viewer открывается |
| P0.3 | Проверить report-v1 для GTRAIN02 | onboarding-cross-node | UI | Низкий | 9 звонков, score ≈78.3, agents не смешаны |

### Команды

```bash
cd /home/DenisErmakov/apps/onboarding_cross_node
node scripts/import_analysis_result.js --file tmp/GTRAIN02_calls_result_FINAL_PRODUCTS.json --dry-run
node scripts/import_analysis_result.js --file tmp/GTRAIN02_calls_result_FINAL_PRODUCTS.json

fuser -k 8020/tcp 2>/dev/null || true
pkill -f "run_forever.sh" 2>/dev/null || true
pkill -f "node.*server" 2>/dev/null || true
nohup ./run_forever.sh > app.log 2>&1 &
sleep 3
tail -n 80 app.log
```

---

## Phase 1 — safety first: семантическая сверка calls

**Цель:** закрыть главный риск перед визуальными правками.

### Задачи

| ID | Задача | Файлы | Риск | Проверка |
|---|---|---|---|---|
| A1 | Добавить semantic-check для `analysis_type=calls` в импорт | `scripts/import_analysis_result.js` | Средний | конфликтный JSON падает в live import |
| A2 | В dry-run показывать semantic warnings/errors | `scripts/import_analysis_result.js` | Низкий | dry-run явно пишет score consistency |
| A3 | Проверять forbidden markers в `call_results`, `question_results`, `stage_dynamics` | validator/import | Средний | `ROLE-*`, `dialog:*`, `result_payload` блокируются |
| A4 | Зафиксировать contract в docs | docs repo | Низкий | новый раздел в docs/20 или отдельный doc |

### Acceptance criteria

- GTRAIN02 проходит dry-run и live import.
- JSON с рассинхроном score не импортируется.
- JSON с учебными агентами в calls не импортируется.
- В stdout dry-run есть понятный блок `Semantic checks`.

---

## Phase 2 — привести `report-v1.html` к v5 UI

**Цель:** live viewer должен выглядеть как целевой v5 HTML.

### Основной принцип

Не копировать весь статический HTML вслепую. Перенести:

- структуру вкладок;
- CSS/визуальные компоненты;
- рендеры блоков;
- управленческие формулировки;
- пустые состояния;
- логику шкал.

Данные должны продолжать приходить из viewer API.

### Задачи

| ID | Задача | Файлы | Риск | Проверка |
|---|---|---|---|---|
| B1 | Обновить hero / верхний блок: опыт, сегмент, направление, даты без времени | `public/report-v1.html` | Низкий | ключ новичка не отображается |
| B2 | Убрать лишнюю «Сводку цифр» и субъективные блоки | `public/report-v1.html` | Низкий | в overview нет status/risk/data readiness |
| B3 | Заменить overview-блок звонков на «Анализ звонков по автомануалу» | `public/report-v1.html` | Средний | нет дубля `Начало/Середина/Выпуск` в overview |
| B4 | Оставить stage dynamics только во вкладке «Звонки» | `public/report-v1.html` | Средний | динамика есть только в calls tab |
| B5 | Перенести шкалы времени на трубке и по дням | `public/report-v1.html` | Средний | 214 мин отображается зелёной/тёмно-зелёной логикой |
| B6 | Исправить карточку «Всего звонков»: жёлтая зона, без технических подписи | `public/report-v1.html` | Низкий | видна только управленческая формулировка |
| B7 | Рекомендации оформить карточками и заменить «Красные флаги» на «Риски» | `public/report-v1.html` | Низкий | UI без `red flags` |
| B8 | Учебные агенты: блоки оценки свернуты по умолчанию | `public/report-v1.html` | Средний | кнопка раскрытия работает |
| B9 | Убрать текстовый фрагмент собеседования из пользовательского UI | `public/report-v1.html` | Низкий | transcript не торчит в overview/interview summary |

### Acceptance criteria

- `node --check` по JS из HTML проходит.
- GTRAIN02 открывается без ошибок.
- Визуально report-v1 совпадает с v5 по структуре и логике.
- Английских пользовательских формулировок нет.
- Блоки не слипаются.

---

## Phase 3 — синхронизировать data contract с v5

**Цель:** UI не должен быть красивой оболочкой над неполными данными.

### Задачи

| ID | Задача | Файлы | Риск | Проверка |
|---|---|---|---|---|
| C1 | Убедиться, что viewer отдаёт `latest_analysis.calls.rubric_result.units` | service/viewer | Средний | overview автомануал строится из 5 блоков |
| C2 | Убедиться, что viewer отдаёт `stage_dynamics` для calls tab | service/viewer | Средний | start/middle/final отображаются |
| C3 | Убедиться, что viewer отдаёт `call_results` для 9 звонков | service/viewer | Средний | 9 отдельных звонков в calls tab |
| C4 | Поддержать `products_detected` в call_results | prompt/import/viewer | Средний | найденные продукты видны в calls tab |
| C5 | Нормализовать call metrics: minutes, calls_total, over_2min_percent, days[] | service/viewer | Средний | шкалы строятся без fallback-мусора |
| C6 | Не показывать технические `source_ref` в пользовательском UI по умолчанию | `public/report-v1.html` | Низкий | source_ref только в свернутых technical details или скрыт |

---

## Phase 4 — промпты, примеры, тесты

**Цель:** Codex/Opus/следующие агенты должны генерировать данные под текущий UI, а не под старую схему.

### Задачи

| ID | Задача | Файлы | Риск | Проверка |
|---|---|---|---|---|
| D1 | Обновить `prompts/codex/calls_analysis_v1.md`: `stage_dynamics`, `call_results`, `products_detected` обязательны | code repo | Средний | prompt явно требует 9 call_results для GTRAIN02 |
| D2 | Обновить `examples/analysis/calls_result_example.json` | code repo | Низкий | пример содержит stage_dynamics/call_results/products |
| D3 | Обновить `scripts/test_rubric_scoring.js` под v1.1.0 | code repo | Средний | 0 failed |
| D4 | Обновить handoff docs после внедрения v5 | docs repo | Низкий | следующий чат понимает актуальное состояние |

---

## Phase 5 — не сейчас, но держать в backlog

Эти задачи важные, но не должны блокировать v5.

| ID | Задача | Почему не сейчас |
|---|---|---|
| E1 | Report snapshots / История отчётов | крупнее, требует БД/генерации HTML |
| E2 | Отдельный `training_agent_analysis_v1` | сначала стабилизировать реальные звонки и v5 UI |
| E3 | Новые вкладки Тестовый день / Погружение / Выпускной тест | scope creep; v5 уже выбрал 7 вкладок |
| E4 | Большой refactor `phase1_candidate_service.js` | полезно, но не перед визуальной стабилизацией |
| E5 | Dashboard BI-функции | после карточки новичка |

---

## 6. Рекомендуемый порядок PR

### PR-1 — Calls semantic import guard

**Тип:** small backend/data safety  
**Цель:** не дать импортировать кривой calls-анализ.  
**Файлы:**

- `scripts/import_analysis_result.js`
- возможно `services/phase1_analysis_result_validator.js`
- тестовый fixture в `tmp`/`examples` без коммита tmp-мусора

**Проверка:**

```bash
node scripts/import_analysis_result.js --file tmp/GTRAIN02_calls_result_FINAL_PRODUCTS.json --dry-run
node scripts/test_rubric_scoring.js
```

### PR-2 — Report v5 overview and calls UI

**Тип:** UI-only / minimal backend if needed  
**Цель:** привести основной отчёт и вкладку «Звонки» к v5.  
**Файлы:**

- `public/report-v1.html`
- минимально viewer service, только если не хватает данных

**Проверка:**

- GTRAIN02 открывается.
- Overview показывает общий автомануал, а не stage dynamics.
- Calls tab показывает stage dynamics и шкалы.
- Нет технических подписей в карточке «Всего звонков».

### PR-3 — Recommendations / training / interview polish

**Тип:** UI polish  
**Цель:** рекомендации, риски, учебные агенты, собеседование.  
**Файлы:**

- `public/report-v1.html`

**Проверка:**

- `Красные флаги` заменены на `Риски`.
- Учебные агенты свернуты по умолчанию.
- Текстовый фрагмент собеседования не торчит в основном UI.

### PR-4 — Prompts/examples/tests sync

**Тип:** docs/test/data contract  
**Цель:** зафиксировать новую схему для будущих анализов.  
**Файлы:**

- `prompts/codex/calls_analysis_v1.md`
- `examples/analysis/calls_result_example.json`
- `scripts/test_rubric_scoring.js`
- docs handoff/status

---

## 7. Smoke checklist для каждого PR

### Базовые проверки

```bash
node --check server.js
node --check onboarding_core.js
node --check sheets_client.js
```

Если менялся HTML с inline JS:

```bash
# извлечь JS из public/report-v1.html и проверить node --check
```

### Проверка GTRAIN02

```bash
node scripts/export_candidate_analysis_bundle.js --base-key GTRAIN02 --type calls --out tmp/GTRAIN02_calls_bundle_CHECK.json
node scripts/import_analysis_result.js --file tmp/GTRAIN02_calls_result_FINAL_PRODUCTS.json --dry-run
```

Ожидаем:

- `real_calls = 9`;
- `start = 3`, `middle = 3`, `final = 3`;
- `product_dictionary > 0`;
- `question_results = 16`;
- `call_results = 9`;
- `overall_score_percent ≈ 78.3`;
- слабая зона — `objections`.

### Проверка UI

Открыть:

```text
http://178.250.243.159:8010/onboarding_cross/report-v1.html?base_key=GTRAIN02
```

Проверить:

- нет ключа новичка в пользовательской части;
- даты без времени;
- опыт кандидата сверху;
- overview calls = общий автомануал;
- stage dynamics только во вкладке «Звонки»;
- 9 звонков;
- продукты отображаются, если есть в `products_detected`;
- учебные агенты не отображаются как реальные звонки;
- блоки не слипаются;
- английского текста нет.

---

## 8. Stop rules

Остановиться и не мержить, если:

- dry-run PASS, но semantic score не сходится;
- `analysis_type=calls` содержит `training_bot_dialogs`, `ROLE-*`, `dialog:*`, `result_payload`;
- после PR исчезли `stage_dynamics` или `call_results`;
- GTRAIN02 показывает не 9 звонков;
- overview снова дублирует `Начало / Середина / Выпуск`;
- UI показывает технические подписи в пользовательских карточках;
- PR трогает слишком много файлов без необходимости;
- PR body противоречит diff.

---

## 9. Короткий backlog

| Priority | ID | Задача | Тип |
|---|---|---|---|
| P0 | A1 | Semantic guard для calls import | backend/data |
| P1 | B1-B7 | Привести `report-v1.html` к v5 overview/calls/recommendations | UI |
| P1 | C1-C5 | Синхронизировать viewer payload с v5 | backend/viewer |
| P1 | D1 | Обновить Codex prompt под обязательные `stage_dynamics/call_results/products_detected` | prompt |
| P2 | D2-D3 | Обновить пример и тесты рубрики | test/docs |
| P2 | C6 | Спрятать technical source_ref из пользовательского UI | UX |
| P3 | E1 | Report snapshots / история отчётов | backend/UX |
| P3 | E2 | training_agent_analysis_v1 | analytics |

---

## 10. Следующий конкретный шаг

Начать с **PR-1: Calls semantic import guard**.

Почему не сразу v5 UI:

- v5 UI будет выглядеть убедительно;
- но если под ним лежит несогласованная математика звонков, отчёт станет опасно убедительным;
- сначала закрываем импортный риск, потом переносим визуал.

После PR-1 делать **PR-2: Report v5 overview and calls UI**.
