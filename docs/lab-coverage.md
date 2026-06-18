# Покрытие лабораторных работ

Эта страница сопоставляет задания из репозитория курса `ITMO_ICT_WebDevelopment_tools_2025-2026` с реальными компонентами Unified Platform. Проект не является набором учебных скриптов, поэтому некоторые требования закрыты промышленными аналогами: фоновые воркеры, async I/O, очереди, контейнеризация и полноценное web-приложение.

## Сводная таблица

| Лабораторная | Требование курса | Реализация в проекте |
| --- | --- | --- |
| ЛР1 | FastAPI-приложение | Backend собран на FastAPI, точка сборки app - `cabinet/api/app.py` |
| ЛР1 | PostgreSQL + ORM | PostgreSQL, SQLAlchemy async models в `shared/db/models.py` |
| ЛР1 | 5+ таблиц и связи | В проекте десятки таблиц: organizations, users, connections, reviews, chats, billing, product transfer |
| ЛР1 | Alembic | Миграции лежат в `unified_platform_backend/alembic/versions/` |
| ЛР1 | CRUD/API | Роутеры для пользователей, команды, подключений, отзывов, вопросов, чатов, биллинга и переноса карточек |
| ЛР1 | Авторизация | OAuth через Yandex ID, сессии в Redis, HttpOnly cookie, RBAC |
| ЛР2 | Асинхронность | FastAPI async endpoints, async SQLAlchemy, httpx AsyncClient, Redis async client |
| ЛР2 | Параллельная обработка | TaskIQ worker обрабатывает фоновые задачи отдельно от HTTP-процесса |
| ЛР2 | Парсинг/получение внешних данных | Интеграции с API маркетплейсов загружают отзывы, вопросы, чаты и карточки товаров |
| ЛР2 | Сохранение результата в БД | Результаты синхронизаций сохраняются в PostgreSQL через репозитории |
| ЛР3 | Dockerfile | Backend и frontend имеют отдельные Dockerfile |
| ЛР3 | Docker Compose | `docker-compose.yml` для PostgreSQL/Redis и `docker-compose.prod.yml` для полного стенда |
| ЛР3 | Redis/очередь/worker | Redis используется как broker/result backend для TaskIQ |
| ЛР3 | API -> очередь -> worker | Reviews, chats, product transfer и billing ставят фоновые задачи, которые выполняет worker |

## ЛР1. FastAPI, БД, CRUD и авторизация

Вместо учебной предметной области реализовано полноценное серверное приложение для управления маркетплейс-операциями.

Основные признаки выполнения:

- FastAPI app создается в `unified_platform_backend/src/unified_platform/cabinet/api/app.py`;
- API разделено на роутеры по предметным областям: cabinet, billing, reviews, questions, chats, product transfer, support;
- SQLAlchemy-модели описаны в `unified_platform_backend/src/unified_platform/shared/db/models.py`;
- миграции Alembic хранят историю схемы БД;
- структура проекта разделена на API, application, domain и infrastructure;
- пользовательский контур включает Yandex OAuth, сессии, роли `admin`/`worker`, приглашения в организацию;
- пароли и marketplace-секреты не отдаются наружу, API-ключи хранятся зашифрованными через Fernet.

Примеры моделей и связей:

| Область | Таблицы и связи |
| --- | --- |
| Организации и пользователи | `organizations` -> `users`, `accounts` -> `users` |
| Подключения | `users`/`organizations` -> marketplace connection tables |
| Отзывы | review tables, comments, reply jobs |
| Чаты | `chat_threads` -> `chat_messages` -> `chat_attachments` |
| Биллинг | payments, transactions, promocodes, redemptions |
| Перенос карточек | processes, drafts, media assets, mappings, product cards |

## ЛР2. Потоки, процессы и асинхронность

В задании ЛР2 требуются примеры threading, multiprocessing и async, а также параллельный парсинг страниц с сохранением в БД. В Unified Platform эта тема покрыта реальной backend-архитектурой:

- HTTP API работает в async runtime FastAPI/Uvicorn;
- доступ к PostgreSQL выполняется через async SQLAlchemy и `asyncpg`;
- внешние API маркетплейсов вызываются через асинхронные HTTP-клиенты;
- Redis используется через async client;
- длительные операции вынесены из HTTP request/response в отдельный worker-процесс;
- worker запускает несколько фоновых циклов: автосинхронизация, dispatch reply jobs, cleanup, Telegram polling поддержки.

Практический аналог “парсинга с сохранением в БД”:

| Учебное требование | Реальный сценарий |
| --- | --- |
| Получить данные из внешнего источника | Синхронизация отзывов, вопросов, чатов и товарных карточек из API маркетплейсов |
| Обработать несколько задач параллельно | TaskIQ worker обрабатывает независимые jobs через Redis queue |
| Сохранить результат | Репозитории сохраняют нормализованные данные в PostgreSQL |
| Сравнить подходы | В проекте выбран async I/O + worker-процесс, потому что операции сетевые и долгие |

Почему выбран такой подход:

- threading плохо подходит как основной механизм для async web-сервиса и усложняет shared state;
- multiprocessing полезен для CPU-bound задач, но интеграции с маркетплейсами в основном I/O-bound;
- async I/O и очередь позволяют не блокировать HTTP-запросы и устойчиво обрабатывать долгие синхронизации.

## ЛР3. Docker, источники данных и очереди

Требования ЛР3 закрыты инфраструктурой проекта.

| Требование | Реализация |
| --- | --- |
| Dockerfile для FastAPI | `unified_platform_backend/Dockerfile` |
| Dockerfile для frontend | `unified_platform_frontend/Dockerfile` |
| PostgreSQL в Compose | `postgres` service в `docker-compose.yml` и `docker-compose.prod.yml` |
| Redis в Compose | `redis` service в `docker-compose.yml` и `docker-compose.prod.yml` |
| Worker в Compose | `worker` service в `docker-compose.prod.yml` |
| Healthcheck | `GET /api/v1/health` |
| Очередь задач | TaskIQ + Redis broker/result backend |

Production-like поток:

```text
Frontend action
  -> FastAPI endpoint
  -> application use case
  -> TaskIQ enqueue
  -> Redis queue
  -> worker process
  -> marketplace API / AI / billing side effect
  -> PostgreSQL update
  -> frontend polling or websocket notification
```

Примеры фоновых задач:

- синхронизация отзывов Wildberries, Ozon и Yandex Market;
- генерация и отправка AI-ответов;
- обработка reply jobs;
- синхронизация вопросов и чатов;
- подготовка черновиков переноса товарных карточек;
- фискализация платежей;
- Telegram polling для support widget.

## Итог

Unified Platform покрывает объем лабораторных не как набор изолированных учебных примеров, а как связанный fullstack-проект:

- FastAPI backend с PostgreSQL, ORM и миграциями;
- async I/O и worker-процессы для долгих операций;
- Redis-backed очередь задач;
- Docker-контейнеризация и production-like compose;
- frontend, который использует backend API и защищенные маршруты.
