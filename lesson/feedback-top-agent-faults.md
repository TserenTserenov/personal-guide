---
name: feedback-top-agent-faults
description: "Топ повторяющихся косяков агента — HOT fault patterns для inject в браузерные сессии"
metadata:
  type: feedback
  horizon: hot
  status: active
  valid_from: 2026-06-03
  owner: user
  schema_version: 1
---

# Топ повторяющихся косяков агента (fault reminders)

Этот файл используется personal_generate_fault_remind для браузерных сессий.
Обновлять при Month Close: sync-fault-reminders.sh + personal_reindex_source.

## 1. Автономность — не задавать вопросов-разрешений (P5, CRITICAL)

Ошибка/fault/косяк: задавать yes/no вопросы «продолжить?», «записать?», «применить?», «хотите?».
Правило: задание → выполни → отчитайся. Без вопросов.
Источник: feedback_behaviour.md, n=853 срабатываний апрель 2026.

## 2. git staging — только конкретные файлы (CRITICAL)

Ошибка/fault/error: git add -u, git add ., git add -A захватывают чужие staged файлы.
Pipe 2>&1 | tail маскирует exit 128 → commit берёт чужие файлы параллельных агентов.
Правило: только git add конкретный-путь. git status --short перед commit.
Источники: feedback_git_add_dash_u.md, lessons_git_add_pipe_exit_masking.md.

## 3. Root cause — верифицировать grep перед публикацией (CRITICAL)

Ошибка/fault/error/bug: публиковать объяснение без grep проверки файл:строка.
Правило: root cause → grep → line number → только потом публиковать.
Источник: feedback_root_cause_verify_in_code.md.

## 4. Финиш > отлог — побочные задачи делать сейчас (P5)

Ошибка/fault: обнаружить задачу и предложить «отдельный РП» или «технический долг».
Правило: дефолт = делаю сейчас. «сейчас или потом» = анти-паттерн.
Источник: feedback_finish_now_no_defer.md.

## 5. Routing Gate — перед Write нового файла (BLOCKING)

Ошибка/fault/bug: класть файл по аналогии без routing-vocab.md.
Правило: перед каждым Write нового файла → открыть routing-vocab.md.
Источник: feedback_routing_gate_always.md.

## 6. Day Open — шаг 5e KE-кандидаты обязателен (CRITICAL)

Ошибка/fault: scaffold без PENDING-маркера → секция 5e молча пропускалась 3 дня подряд.
Правило: KE-кандидаты — обязательный шаг Day Open.
Источник: feedback_day_open_captures_skipped.md.

## 7. Peer adapter — negative scope в промпте (CRITICAL)

Ошибка/fault/error: claude-peer-adapter.sh без «НЕ редактировать файлы» → Claude self-execute.
Правило: peer-реплика через адаптер обязана иметь negative scope.
Источник: feedback_peer_adapter_role_overreach.md.

## 8. Именование РП — существительное-артефакт не глагол (BLOCKING)

Ошибка/fault: «Разработать систему», «Настроить MCP» — глаголы в названии РП.
Правило: название РП = существительное-артефакт. REGISTRY ≤80 символов.
Источник: feedback_wp_naming_verbs.md.

## 9. Kimi читает AGENTS.md, не CLAUDE.md (HOT)

Ошибка/fault/bug: инструкции в CLAUDE.md — Kimi игнорирует.
Правило: для Kimi → AGENTS.md. Для Claude → CLAUDE.md.
Источник: feedback_kimi_agents_md.md.

## 10. Python triple-quote SQL trap (CRITICAL)

Ошибка/fault/error/bug: Python пожирает '' в triple-quote → SQL обрывается → runtime crash.
Правило: length(col) > 0 вместо != '' в Python triple-quote SQL.
Источник: lessons_python_triple_quote_sql_trap.md.

## 11. WP Gate — grep WP-REGISTRY ДО meta.yaml (HOT)

Ошибка/fault: task_id = archived WP вместо активного.
Правило: grep WP-REGISTRY ДО создания meta.yaml сессии.
Источник: feedback_wp_gate_wrong_task_id.md.

## 12. Close — git status по ВСЕМ репо ДО отчёта (BLOCKING)

Ошибка/fault/bug: «закрывай» без git status → незалитые изменения после отчёта.
Правило: при Close — обойти все ~/IWE/* репо, push ДО отчёта.
Источник: feedback_close_push_all_repos.md.
