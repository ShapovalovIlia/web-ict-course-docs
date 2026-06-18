# Эксплуатация

Проект поддерживает локальный запуск для разработки и production-like запуск через Docker Compose.

## Локальная инфраструктура

Для разработки backend достаточно поднять PostgreSQL и Redis:

```bash
cd unified_platform_backend
docker-compose up
```

Сервисы:

| Сервис | Назначение |
| --- | --- |
| `postgres` | основная PostgreSQL БД |
| `redis` | сессии, OAuth state, rate limit, TaskIQ broker/result backend |

## Production-like Compose

`unified_platform_backend/docker-compose.prod.yml` описывает полный контур:

| Сервис | Назначение |
| --- | --- |
| `postgres` | основная БД |
| `redis` | кэш, сессии и очередь |
| `backend` | FastAPI API |
| `frontend` | production frontend image |
| `worker` | TaskIQ worker |
| `loki` | сбор логов |
| `grafana` | дашборды и алерты |
| `alloy` | доставка логов в Loki |

Backend image при старте выполняет миграции и запускает Uvicorn:

```text
uv run --no-sync alembic upgrade head
uv run --no-sync uvicorn unified_platform.main:app --host 0.0.0.0 --port 8000
```

Worker запускается отдельным процессом:

```text
uv run --no-sync taskiq worker unified_platform.reviews.infrastructure.tasks.worker:broker
```

## Healthcheck

Минимальная проверка backend:

```bash
curl http://127.0.0.1:8000/api/v1/health
```

Ожидаемый результат - успешный HTTP-ответ. В production-like compose healthcheck backend использует этот endpoint, а frontend дополнительно проверяет доступность `/` и `/api/v1/health`.

## CI/CD и staging

Основной маршрут проверки на тестовом стенде:

| Изменение | Маршрут |
| --- | --- |
| Backend/fullstack | commit/push в `unified_platform_backend` -> GitLab pipeline -> deploy staging -> smoke |
| Frontend | commit/push в `unified_platform_frontend` -> сборка frontend image -> trigger backend staging deploy |

Для обычных задач предпочтителен CI/CD deploy, а не ручной SSH patch-deploy.

## Observability

Проект пишет структурированные JSON logs. В production-like окружении используются:

- Loki для хранения логов;
- Grafana для просмотра и алертов;
- Alloy для сбора логов контейнеров;
- healthchecks Docker Compose для контроля базовой доступности.

Отдельные документы backend:

- `unified_platform_backend/docs/deployment-environments.md`;
- `unified_platform_backend/docs/observability.md`;
- `reference/prompts/production-log-audit.md`.

## Минимальный smoke checklist

- `GET /api/v1/health` отвечает успешно;
- frontend открывается без runtime error;
- `/users/me` корректно отрабатывает для авторизованной сессии;
- protected route не ломает redirect/auth flow;
- для admin доступны команда, подключения и биллинг;
- для worker недоступны admin-only действия;
- фоновые задачи проходят путь API -> Redis -> TaskIQ worker -> PostgreSQL update.
