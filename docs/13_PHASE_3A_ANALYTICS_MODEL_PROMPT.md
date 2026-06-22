# PHASE_3A_ANALYTICS_MODEL_PROMPT.md

Промпт для Codex / Claude. Использовать только после подтверждения.

```text
Сделай Phase 3A — Analytics Model & Rubrics audit/design.

Контекст:
Phase 1/2/2B дали серверное хранилище, импорты, upload файлов и админку сборки данных. Но целевой продукт — dashboard новичков, аналитическая карточка, scores, рекомендации и HTML-отчёт.

Перед кодом нужно зафиксировать, как аналитика будет храниться и выводиться.

Прочитай документы:
- 11_EVALUATION_RUBRICS_BY_STAGE_V1.md
- 12_ANALYTICS_DASHBOARD_AND_REPORT_SPEC_V1.md
- 05_AI_ANALYSIS_AND_SCORING.md
- 06_DASHBOARD_SPEC.md
- 08_ROADMAP.md

Сделай audit/design only. Код не меняй.

Ответь:
1. Какие данные уже есть в SQLite.
2. Какие таблицы нужны для аналитики.
3. Какие manual sections нужны.
4. Какие scores должны быть в candidate_scores.
5. Какие endpoints нужны для admin analytics.
6. Какие endpoints нужны для viewer dashboard.
7. Как будет выглядеть карточка новичка.
8. Как будет выглядеть dashboard.
9. Как хранить HTML report snapshots.
10. Какие риски.
11. План Phase 3B/3C/3D.
12. Какие файлы потенциально менять.

В конце напиши:
Жду команду Делай.
```
