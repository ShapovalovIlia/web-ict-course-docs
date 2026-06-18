# Unified Platform

**Студент:** Шаповалов Илья Андреевич  
**Группа:** К3339  
**Дисциплина:** Средства Web-программирования  
**Проект:** Unified Platform

Unified Platform - это платформа для управления отзывами, вопросами, чатами, подключениями маркетплейсов, балансом организации и переносом товарных карточек между маркетплейсами.

Документация подготовлена как минимальный отчет по проекту, который был показан преподавателю вместо отдельных лабораторных работ. Основная цель - показать, какие части реального fullstack-проекта покрывают объем заданий ЛР1-ЛР3.

## Стек

| Слой | Технологии |
| --- | --- |
| Backend | Python 3.13, FastAPI, Pydantic, Dishka |
| База данных | PostgreSQL, SQLAlchemy 2.x asyncio, Alembic |
| Очереди и кэш | Redis, TaskIQ, taskiq-redis |
| Frontend | React, TypeScript, Vite, Tailwind CSS |
| Инфраструктура | Docker, Docker Compose, GitLab CI/CD |
| Наблюдаемость | JSON logs, Loki, Grafana, Alloy |

## Состав монорепо

```text
unified_platform/
├── unified_platform_backend/    # FastAPI + PostgreSQL + Redis + TaskIQ
├── unified_platform_frontend/   # React + TypeScript + Vite + Tailwind CSS
├── docs/                        # эта учебная и проектная документация
├── reports/                     # отчеты по изменениям
├── reference/                   # внешние API, планы, аудит-промпты
└── archive/                     # legacy-код, не используется в текущей сдаче
```

Backend и frontend живут как отдельные git-проекты, а корень используется как рабочая оболочка монорепо и место общей документации.

## Основные возможности проекта

- авторизация через Yandex ID и server-side cookie-сессии в Redis;
- организации, роли `admin` и `worker`, приглашения в команду;
- подключения маркетплейсов Wildberries, Yandex Market, Ozon, Magnit Market, Avito, MegaMarket, M.Video;
- работа с отзывами, вопросами и чатами маркетплейсов;
- AI-ответы, автоответы и фоновые reply jobs;
- баланс организации, транзакции, промокоды, пополнение и фискализация;
- перенос товарных карточек между маркетплейсами;
- production-like Docker Compose с backend, frontend, worker, PostgreSQL, Redis и observability.

## Быстрый просмотр документации

Локальный запуск:

```bash
uvx --with mkdocs-material mkdocs serve -a 127.0.0.1:8001
```

Статическая сборка:

```bash
uvx --with mkdocs-material mkdocs build
```

## Где смотреть код

| Что нужно проверить | Путь |
| --- | --- |
| Точка входа backend | `unified_platform_backend/src/unified_platform/main.py` |
| Сборка FastAPI app | `unified_platform_backend/src/unified_platform/cabinet/api/app.py` |
| SQLAlchemy-модели | `unified_platform_backend/src/unified_platform/shared/db/models.py` |
| Миграции | `unified_platform_backend/alembic/versions/` |
| TaskIQ worker | `unified_platform_backend/src/unified_platform/reviews/infrastructure/tasks/worker.py` |
| Frontend routes | `unified_platform_frontend/src/App.tsx` |
| API-клиент frontend | `unified_platform_frontend/src/api/client.ts` |
