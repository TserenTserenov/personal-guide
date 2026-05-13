---
name: lesson-close
description: Закрыть занятие, открытое скиллом /lesson. Финализирует workbook/YYYY-MM-DD.md (frontmatter status, метаданные времени, длительность), делает commit + push в репо personal-guide. Триггерит замкнутый контур доставки — после push → GitHub webhook → bot oauth_server.py:/webhook/github/workbook → sync_one_user_to_dt → ЦД обновляется в Neon. Используй когда пилот говорит «закрываем», «всё», «закончили», «закрой урок» — или после явного завершения задания в /lesson.
argument-hint: "[необязательно: дата урока YYYY-MM-DD; по умолчанию сегодня; либо --skipped если урок был пропущен; --no-push для локального commit без push]"
experimental: true
sunset: "после WP-301 Ф6 (E2E smoke) и WP-149 Block D (ИИ-агент-носитель Портного)"
related: [WP-149, WP-175, WP-245, WP-301, lesson, PD.METHOD.008]
---

# /lesson-close — закрыть занятие и замкнуть контур доставки

> ⚡ **Алгоритм, не диалог.** Шаги 1-5 последовательно. Без вопросов.

## Контракт скилла

- **Вход:** существующий `workbook/YYYY-MM-DD.md` в репо `personal-guide` со статусом `status: in_progress` (создан скиллом `/lesson`). Активный git remote с правом push.
- **Выход:** workbook-файл с финализированным frontmatter (`status: done|partial|skipped`, `finished_at`, `duration_min`) + git commit + git push. На сервере: webhook triggered → `sync_one_user_to_dt(user_id)` → Neon `digital_twins.data['2_collected']` обновлён + событие в `development.user_events`.
- **Время:** ≤2 мин (быстрая финализация после `/lesson`).
- **Не делает:** не оценивает качество ответа (Оценщик R12 — отдельно, асинхронно); не генерирует assignment на завтра (Портной, ночной рендер).

## Шаг 1. Найти и прочитать workbook-файл

1. Определи путь: `workbook/YYYY-MM-DD.md`. Дата = аргумент скилла или сегодня.
2. Если файл не существует — сообщи пилоту: «Workbook на эту дату не найден. Сначала запусти `/lesson` или укажи правильную дату.» — стоп.
3. Прочитай frontmatter и тело файла.

## Шаг 2. Определить статус занятия

Логика по умолчанию (если не передан `--skipped`):

| Условие | Статус |
|---|---|
| Есть ответ на «Вопрос» И есть ответ/прогресс на «Задание» | `done` |
| Есть ответ на «Вопрос», но задание помечено `to-do` или пустое | `partial` |
| Нет ответа на «Вопрос» И нет на «Задание» | `skipped` |
| Аргумент `--skipped` передан явно | `skipped` (override) |

Спросй пилота **только если статус неоднозначен** (например, задание имеет частичный ответ — `partial` vs `done`?). Одно сообщение, два варианта: «Считаем done или partial? (one word)».

## Шаг 3. Финализировать frontmatter

Обнови frontmatter workbook-файла:

```yaml
---
date: YYYY-MM-DD
assignment_ref: daily/YYYY-MM-DD.md  # или assignments/ для legacy
theme: <existing>
element_id: <existing>
stage: <existing>
bottleneck: <existing>
started_at: <existing>
finished_at: <ISO timestamp now, UTC>
duration_min: <разница finished_at - started_at в минутах>
status: done|partial|skipped
---
```

Используй `Edit` для обновления только frontmatter; тело файла не трогать.

## Шаг 4. Git commit + push

Через Bash в каталоге репо `personal-guide`:

```bash
git add workbook/YYYY-MM-DD.md
git commit -m "lesson YYYY-MM-DD: <theme> (status: <done|partial|skipped>)"
git push
```

**Если `--no-push` передан** — пропусти push. Это локальный режим (например, для теста).

**Если push fails** (auth, network, conflict) — сообщи пилоту: «Commit готов локально, push не прошёл: <ошибка>. Запусти `git push` вручную, когда сеть будет.» Контур всё равно замкнётся, как только пилот сделает push.

## Шаг 5. Сообщить пилоту о замыкании контура

После успешного push:

```
✅ Урок закрыт.
Статус: <status>
Длительность: <duration_min> мин
Коммит: <short_sha>

Контур замкнут:
  workbook/<date>.md → git push → GitHub webhook → Activity Hub → Neon (ЦД)
  Завтрашний assignment Портной отрендерит на свежих данных.
```

Не нужно ждать или проверять, что webhook реально сработал. Он асинхронный, факт push — единственное, что от тебя требуется. Если интересно: проверь логи `[WorkbookWebhook]` в Railway аист-боте.

## Граничные случаи

| Ситуация | Действие |
|---|---|
| Workbook со статусом `done` уже (повторный close) | Сообщи: «Урок уже закрыт. Если нужно переоткрыть — измени status: in_progress вручную или удали finished_at.» |
| Нет git remote / нет прав push | Сообщи и предложи проверить `git remote -v` и `gh auth status`. Финальный коммит локально остаётся. |
| Merge-conflict при push | НЕ разрешать автоматически. Сообщи пилоту: «Кто-то ещё пушил в этот репо. Проверь: `git pull --rebase` и реши конфликт.» |
| Очень короткое занятие (<3 мин) | Записать как обычно, не оценивать. Поле `duration_min` уйдёт в метрику. |
| Длинное занятие (>60 мин) | То же. Метрика покажет реальное время. |
| Пилот хочет сменить тему текущего занятия задним числом | Не редактируй `theme` или `element_id` — это нарушает соответствие assignment ↔ workbook. Если нужно — открой новый workbook вручную. |

## Связь с другими механизмами

- **`/lesson`** — парный скилл, открывает занятие, создаёт workbook-скелет.
- **GitHub webhook** — handler в `aist_bot_newarchitecture/oauth_server.py:1183` (роут `POST /webhook/github/workbook`). HMAC-секрет = `GITHUB_WORKBOOK_WEBHOOK_SECRET` в Railway env. Регистрируется один раз в `personal-guide` репо: Settings → Webhooks.
- **Активность в Neon:** webhook записывает событие в `development.user_events` (source=`iwe`, event_type=`workbook_push`) и триггерит `sync_one_user_to_dt(user_id)` → пишет в `digital_twins.data['2_collected']`.
- **Профайлер (асинхронно):** `DS-ai-systems/profiler/scripts/dt_calc.py` пересчитывает `3_derived` на основе обновлённого `2_collected`. Запускается отдельным cron.
- **Портной (следующий рендер):** утром скилл `/personal-guide-render` (или ИИ-агент-носитель WP-149 Block D) читает свежий ЦД → новый `daily/<завтра>.md`.

## Антипаттерны

- ❌ Не делать commit без push, если `--no-push` не передан — контур не замкнётся
- ❌ Не редактировать тело workbook-файла (только frontmatter) — ответы пилота неприкосновенны
- ❌ Не пересоздавать workbook при повторном close — выйдет дубликат истории
- ❌ Не разрешать merge-conflict автоматически — может затереть незакоммиченные ответы
- ❌ Не «ждать ответ webhook» — он асинхронный, неблокирующий
