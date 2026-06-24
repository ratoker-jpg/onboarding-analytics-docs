# New chat handoff — onboarding calls analysis

Дата фиксации: 24.06.2026  
Назначение: промпт для нового чата, чтобы продолжить работу без потери контекста.

---

## Короткий промпт для нового чата

Скопируй текст ниже в новый чат.

```text
Ты продолжаешь работу над проектом аналитики новичков для кросс-продаж.

Главный репозиторий кода:
ratoker-jpg/onboarding-cross-node

Репозиторий документации:
ratoker-jpg/onboarding-analytics-docs

Сервер:
178.250.243.159
Пользователь:
DenisErmakov
Рабочая папка:
/home/DenisErmakov/apps/onboarding_cross_node
Публичный префикс:
/onboarding_cross/
Внутренний порт:
8020

Стандартный restart:
cd /home/DenisErmakov/apps/onboarding_cross_node
fuser -k 8020/tcp 2>/dev/null || true
pkill -f "run_forever.sh" 2>/dev/null || true
pkill -f "node.*server" 2>/dev/null || true
nohup ./run_forever.sh > app.log 2>&1 &
sleep 3
tail -n 80 app.log

Текущий тестовый кандидат:
base_key = GTRAIN02
ФИО = Иванов Иван
Сегмент = Грузоперевозки
Направление = S

Ключевой контекст:
Мы строим механику анализа реальных звонков новичка. Раньше система смешивала реальные звонки с учебными агентами и считала 1 transcript-файл = 1 звонок. Это исправлено.

Текущая правильная цепочка:
1. calls export берёт только реальные звонки из calls_start / calls_middle / calls_final.
2. Учебные агенты/training_bot_dialogs/ROLE/dialog/result_payload запрещены для analysis_type=calls.
3. transcript-файлы нарезаются на отдельные звонки: real_calls[].
4. Для GTRAIN02 smoke показал real_calls: 9 — start=3, middle=3, final=3.
5. Рубрика calls_automanual_binary_v1 обновлена до v1.1.0.
6. В рубрике 16 вопросов: contact 2, needs 4, presentation 4, objections 4, close 2.
7. Вопросы presentation и objections должны использовать product_dictionary.
8. product_dictionary подтягивается из Google Sheet:
   https://docs.google.com/spreadsheets/d/1grwKJPJ3VH6OE0Ky5v3J4FZVADHm7tJrFPwoppkdakI/edit?gid=0#gid=0
9. Берутся все продукты, кроме явно не продающихся: Не продается / Не продаётся / Не продают.
10. Названия продуктов чистятся от служебных пометок “Отредактировано ...”; raw_product_name сохраняется.
11. Smoke по GTRAIN02 показал product_dictionary: 39, dirty_product_names: 0, excluded_status_leaks: 0.
12. Codex сделал результат tmp/GTRAIN02_calls_result_FINAL_PRODUCTS.json.
13. Сначала он дал call_quality_score 88, но это было несогласованно с call_results.
14. Результат исправили: dry-run теперь PASS, call_quality_score = 78.3, средняя по call_results = 78.33, question_results = 16.

Важное правило:
Не импортировать результат, если top-level call_quality_score не согласован с call_results и stage_dynamics. Dry-run проверяет форму JSON, но не смысловую математику.

Актуальный файл результата:
tmp/GTRAIN02_calls_result_FINAL_PRODUCTS.json

Known next step:
Если результат ещё не импортирован, выполнить:
cd /home/DenisErmakov/apps/onboarding_cross_node
node scripts/import_analysis_result.js --file tmp/GTRAIN02_calls_result_FINAL_PRODUCTS.json

Потом перезапустить сервер и проверить:
http://178.250.243.159:8010/onboarding_cross/report-v1.html?base_key=GTRAIN02

Ожидаем во вкладке “Звонки”:
- call quality около 78.3;
- stage_dynamics: Начало / Середина / Выпуск;
- 9 отдельных звонков;
- найденные продукты в call_results;
- слабая зона: отработка возражений;
- учебные агенты не отображаются как реальные звонки.

Документация, которую нужно держать под рукой:
- docs/19_CALLS_ANALYSIS_STATUS_20260624.md
- docs/18_RICH_REPORT_SCOPE_AND_TRAINING_AGENTS_DECISIONS.md
- docs/11_EVALUATION_RUBRICS_BY_STAGE_V1.md
- docs/12_ANALYTICS_DASHBOARD_AND_REPORT_SPEC_V1.md

Когда отвечаешь пользователю:
- говори коротко и по делу;
- не предлагай новый большой PR, если нужен маленький fix;
- перед merge проверяй scope, changed files, smoke и противоречия в PR body;
- не запускай Codex-анализ, пока bundle не содержит real_calls[] + rubric v1.1.0 + product_dictionary[];
- не импортируй анализ, пока score не совпадает с детализацией.
```

---

## Полный чек-лист продолжения

### 1. Проверить, импортирован ли финальный calls result

```bash
cd /home/DenisErmakov/apps/onboarding_cross_node
node scripts/import_analysis_result.js \
  --file tmp/GTRAIN02_calls_result_FINAL_PRODUCTS.json \
  --dry-run
```

Ожидание:

```text
Validation: PASS
call_quality_score: 78.3
```

Если dry-run PASS и score согласован, импорт:

```bash
node scripts/import_analysis_result.js \
  --file tmp/GTRAIN02_calls_result_FINAL_PRODUCTS.json
```

### 2. Перезапуск сервера

```bash
fuser -k 8020/tcp 2>/dev/null || true
pkill -f "run_forever.sh" 2>/dev/null || true
pkill -f "node.*server" 2>/dev/null || true
nohup ./run_forever.sh > app.log 2>&1 &
sleep 3
tail -n 80 app.log
```

### 3. Проверка отчёта

Открыть:

```text
http://178.250.243.159:8010/onboarding_cross/report-v1.html?base_key=GTRAIN02
```

Сделать `Ctrl + F5`.

Проверить вкладку «Звонки»:

```text
score ≈ 78.3
stage_dynamics есть
call_results = 9
products_detected показываются
учебные агенты не смешиваются со звонками
```

### 4. Если отчёт показывает 88

Проверить, какой analysis_run берёт viewer, и что импорт реально создал новую запись.

Не делать новый анализ сразу. Сначала проверить:

```bash
node - <<'NODE'
const fs = require('fs');
const r = JSON.parse(fs.readFileSync('tmp/GTRAIN02_calls_result_FINAL_PRODUCTS.json', 'utf8'));
const payload = r.output_payload || r.output_payload_json || r;
const calls = payload.call_results || [];
const avg = calls.reduce((a,c)=>a+Number(c.overall_percent||0),0)/Math.max(calls.length,1);
console.log('avg call_results:', Math.round(avg*100)/100);
console.log('call_results count:', calls.length);
console.log('question_results count:', (payload.question_results || r.question_results || []).length);
NODE
```

### 5. Если products_detected не видно в UI

Проверить viewer projection и `public/report-v1.html`: поле `call_results[].products_detected` должно доходить до фронта и показываться в таблице «Разбор отдельных звонков».
