# Архитектура

Unified Platform построен как fullstack-приложение с отдельными backend и frontend проектами. Backend следует Clean Architecture: бизнес-логика отделена от FastAPI, SQLAlchemy, Redis и внешних API.

## Backend

Основные слои:

| Слой | Ответственность |
| --- | --- |
| `api/` | FastAPI routers, Pydantic schemas, dependencies, error handlers |
| `application/` | use cases, DTO, orchestration, transactions |
| `domain/` | модели предметной области, value objects, исключения, порты |
| `infrastructure/` | SQLAlchemy repositories, Redis, HTTP clients, TaskIQ, DI |
| `shared/` | общие настройки, БД, middleware, security, logging, rate limit |

Модули backend:

| Модуль | Назначение |
| --- | --- |
| `cabinet` | пользователи, организации, роли, приглашения, подключения маркетплейсов |
| `reviews` | отзывы, AI-ответы, автоответы, reply jobs, websocket events |
| `questions` | вопросы маркетплейсов и ответы на них |
| `chats` | чаты маркетплейсов и синхронизация сообщений |
| `billing` | баланс организации, платежи, транзакции, промокоды, фискализация |
| `product_transfer` | перенос товарных карточек и подготовка черновиков |
| `support` | support widget и Telegram-интеграция |

## Точки входа

| Компонент | Путь |
| --- | --- |
| FastAPI entrypoint | `unified_platform_backend/src/unified_platform/main.py` |
| Создание app | `unified_platform_backend/src/unified_platform/cabinet/api/app.py` |
| DI container | `unified_platform_backend/src/unified_platform/shared/di/container.py` |
| SQLAlchemy models | `unified_platform_backend/src/unified_platform/shared/db/models.py` |
| RBAC dependencies | `unified_platform_backend/src/unified_platform/cabinet/api/dependencies/rbac.py` |
| TaskIQ worker | `unified_platform_backend/src/unified_platform/reviews/infrastructure/tasks/worker.py` |

## Данные и мультитенантность

Главная единица владения данными - организация. Пользователь относится к организации, а баланс также принадлежит организации, а не отдельному пользователю.

Ключевые правила:

- данные отзывов, вопросов, чатов, биллинга и переноса карточек ограничиваются `organization_id`;
- роли `admin` и `worker` проверяются на backend через dependencies;
- admin управляет командой, подключениями и балансом;
- worker работает с операционными разделами без admin-only действий;
- секреты маркетплейсов хранятся зашифрованными через Fernet.

## Фоновые задачи

Длительные операции не выполняются внутри HTTP-запроса. Для них используется связка TaskIQ + Redis:

```text
FastAPI endpoint
  -> use case
  -> scheduler/enqueue
  -> Redis queue
  -> TaskIQ worker
  -> repository/external API
  -> PostgreSQL
```

Так обрабатываются синхронизация отзывов, ответы, чаты, вопросы, перенос карточек, фискализация и support polling.

## Frontend

Frontend реализован на React + TypeScript + Vite.

Основные части:

| Путь | Назначение |
| --- | --- |
| `src/App.tsx` | главный роутинг |
| `src/components/ProtectedRoute.tsx` | проверка сессии и защищенные маршруты |
| `src/components/AdminRoute.tsx` | admin-only guard |
| `src/hooks/useCurrentUser.ts` | текущий пользователь и роль |
| `src/api/client.ts` | базовый API-клиент и обработка 401 |
| `src/pages/` | страницы кабинета, отзывов, команды, биллинга, переноса карточек |

Frontend не является отдельной учебной лабораторной, но показывает практическое использование backend API, авторизации, RBAC и fullstack-сценариев.
