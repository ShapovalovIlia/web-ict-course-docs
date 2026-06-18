# Разработка

Эта страница фиксирует минимальный набор команд и точек входа, которые нужны для проверки проекта и документации.

## Backend

Установка зависимостей:

```bash
cd unified_platform_backend
uv sync
```

Локальная инфраструктура:

```bash
cd unified_platform_backend
docker-compose up
```

Миграции:

```bash
cd unified_platform_backend
uv run alembic upgrade head
```

Запуск API:

```bash
cd unified_platform_backend
uv run uvicorn unified_platform.main:app --reload
```

По умолчанию backend доступен на `http://127.0.0.1:8000`.

## Worker

Worker нужен для фоновых задач: синхронизация отзывов, вопросов, чатов, автоответы, reply jobs, перенос карточек и фискализация.

```bash
cd unified_platform_backend
uv run taskiq worker unified_platform.reviews.infrastructure.tasks.worker:broker
```

TaskIQ broker и result backend используют Redis.

## Frontend

Установка зависимостей и запуск:

```bash
cd unified_platform_frontend
npm install
npm run dev
```

По умолчанию frontend доступен на `http://localhost:3000`.

## Проверки

Backend:

```bash
cd unified_platform_backend
uv run ruff format .
uv run ruff check .
uv run mypy .
uv run pytest
```

Frontend:

```bash
cd unified_platform_frontend
npm run lint
npm test
npm run build
```

Документация:

```bash
uvx --with mkdocs-material mkdocs build
```

## Основные файлы для разработки

| Задача | Где смотреть |
| --- | --- |
| Добавить endpoint | `*/api/routers/` нужного backend-модуля |
| Добавить use case | `*/application/use_cases/` |
| Изменить модель БД | `shared/db/models.py` и новая Alembic migration |
| Подключить dependency | `shared/di/container.py` и providers нужного модуля |
| Добавить фоновые задачи | `*/infrastructure/tasks/` и worker registration |
| Изменить frontend API | `unified_platform_frontend/src/api/` |
| Изменить страницу | `unified_platform_frontend/src/pages/` |

## Правила изменений

- Сначала смотреть существующий паттерн в модуле, затем писать код в том же стиле.
- Backend API, frontend API-клиент и UI должны меняться согласованно.
- Любое изменение схемы БД требует Alembic migration.
- Org-scoped данные должны фильтроваться по организации.
- Admin-only действия защищаются на backend.
- Секреты нельзя логировать, возвращать в API или хранить без шифрования.
- `archive/` не используется для текущей разработки.
