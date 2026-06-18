# API

Backend API доступен под префиксом `/api/v1`. Полный контракт можно смотреть через OpenAPI FastAPI в запущенном приложении, а здесь приведена краткая карта групп endpoint'ов.

## Общие принципы

- Авторизация основана на server-side session cookie.
- Сессии хранятся в Redis.
- Защищенные endpoint'ы получают текущего пользователя через backend dependencies.
- Admin-only операции защищаются на backend, frontend-скрытие кнопок не считается достаточной защитой.
- Данные должны быть ограничены организацией пользователя.

## Группы endpoint'ов

| Группа | Примерный путь | Доступ | Назначение |
| --- | --- | --- | --- |
| Health | `GET /api/v1/health` | public | Проверка готовности backend |
| Auth | `/api/v1/auth/*` | public/session | OAuth flow, callback, logout |
| Users | `/api/v1/users/*` | authenticated | Текущий профиль, аватар, удаление аккаунта |
| Team | `/api/v1/team/*` | admin | Участники организации, роли, управление командой |
| Invites | `/api/v1/invites/*` | admin/public accept | Приглашения в организацию |
| Marketplace connections | `/api/v1/wildberries/*`, `/ozon/*`, `/yandex-market/*`, `/megamarket/*` | admin | Создание, чтение, ротация и отзыв подключений |
| Reviews | `/api/v1/reviews/*` | admin/worker | Список отзывов, детали, ответы, AI-операции |
| Reviews websocket | `/api/v1/reviews/ws/*` | authenticated | События синхронизации и reply jobs |
| Questions | `/api/v1/questions/*` | admin/worker | Вопросы маркетплейсов и ответы |
| Chats | `/api/v1/chats/*` | admin/worker | Чаты маркетплейсов и сообщения |
| Billing | `/api/v1/billing/*` | admin | Баланс, транзакции, пополнение, промокоды |
| Webhook | billing webhook path | external provider | Обработка платежных callback'ов |
| Product transfer | `/api/v1/product-transfer/*` | admin/worker по сценарию | Карточки товаров, процессы переноса, черновики |
| Support | `/api/v1/support/*` | authenticated/public widget parts | Support widget и Telegram-связка |
| Notifications | `/api/v1/notifications/*` | authenticated | Уведомления пользователя |

## Примеры пользовательских сценариев

### Авторизация

```text
Пользователь открывает frontend
  -> переходит в Yandex OAuth
  -> backend проверяет callback и state
  -> backend создает/находит пользователя
  -> session_id сохраняется в Redis
  -> HttpOnly cookie устанавливается в браузер
  -> frontend вызывает /users/me
```

### Подключение маркетплейса

```text
Admin вводит API-ключ
  -> frontend отправляет запрос в marketplace endpoint
  -> backend валидирует токен через внешний API
  -> секрет шифруется Fernet
  -> подключение сохраняется в PostgreSQL
```

### Синхронизация отзывов

```text
Пользователь запускает синхронизацию
  -> FastAPI endpoint ставит задачу
  -> TaskIQ кладет job в Redis
  -> worker получает job
  -> worker обращается к API маркетплейса
  -> результат сохраняется в PostgreSQL
  -> UI получает обновления через polling/websocket
```

### Биллинг

```text
Admin создает пополнение
  -> backend создает payment и transaction
  -> провайдер присылает webhook
  -> backend проверяет событие
  -> баланс организации обновляется
  -> фискализация выполняется отдельной фоновой задачей
```

## Роли

| Функция | admin | worker |
| --- | --- | --- |
| Просмотр отзывов, вопросов и чатов | да | да |
| Ответы на отзывы и вопросы | да | да |
| Управление командой | да | нет |
| Подключение/отзыв маркетплейсов | да | нет |
| Баланс и пополнение | да | нет |
| Операционные задачи переноса карточек | да | частично, по backend policy |

## Где смотреть реализацию

| Что | Путь |
| --- | --- |
| Подключение роутеров | `unified_platform_backend/src/unified_platform/cabinet/api/app.py` |
| Cabinet routers | `unified_platform_backend/src/unified_platform/cabinet/api/routers/` |
| Reviews routers | `unified_platform_backend/src/unified_platform/reviews/api/routers/` |
| Billing routers | `unified_platform_backend/src/unified_platform/billing/api/routers/` |
| Product transfer routers | `unified_platform_backend/src/unified_platform/product_transfer/api/routers/` |
| Frontend API modules | `unified_platform_frontend/src/api/` |
