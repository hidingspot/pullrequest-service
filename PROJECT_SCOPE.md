# Strong Junior Go Project: Pull Request Automation Service

## 1. Цель проекта

Сделать микросервис на Go, который:

1. Принимает HTTP-запрос на создание pull request.
2. Создает pull request в Git-платформе.
3. Назначает reviewer'а.
4. Отслеживает статус review через webhook.
5. Обновляет внутренний статус pull request.

Git-платформу фиксируем сразу: **только GitHub**.

Это должен быть не "игрушечный CRUD", а небольшой, но цельный backend-сервис с внешней интеграцией, понятной архитектурой и проверяемым поведением.

## 2. На что целимся

Цель не в том, чтобы покрыть все сценарии GitHub/GitLab, а в том, чтобы показать strong junior уровень:

- понятный API и структура проекта;
- аккуратная работа с внешним API;
- разделение слоев `handler -> service -> client/repository`;
- конфигурация через env;
- логирование;
- обработка ошибок;
- unit-тесты на ключевую бизнес-логику;
- документация и запуск через Docker Compose.

Итог: проект должен выглядеть как сервис, который можно показать на собеседовании и по которому видно инженерную дисциплину.

## 3. Правильный scope

Для strong junior проекта сразу фиксируем **одну платформу: GitHub**.

Почему это правильный выбор:

- scope не расползается;
- можно сделать интеграцию аккуратно;
- легче тестировать и документировать;
- проект выглядит завершенным, а не недоделанным мультипровайдером.

### Что не делаем в первой версии

Не включаем в MVP:

- поддержку GitLab или других платформ;
- UI;
- Kafka/RabbitMQ "просто чтобы было";
- сложную auth-модель с ролями пользователей;
- работу с реальными git-репозиториями локально;
- автоматический merge;
- автоконфликты merge, ребейз, повторные approvals и прочие edge-case'ы enterprise-уровня.

## 4. Формулировка проекта

Рабочая формулировка:

> HTTP-микросервис на Go для автоматизации pull request workflow в GitHub: создание PR, назначение reviewer'а, прием webhook-событий review и обновление внутреннего статуса pull request.

Это описание уже достаточно конкретное и не размытое.

## 5. Пользовательский сценарий

Базовый сценарий должен быть таким:

1. Клиент отправляет запрос в сервис.
2. Сервис валидирует входные данные.
3. Сервис вызывает GitHub API и создает pull request.
4. Сервис назначает reviewer'а.
5. GitHub присылает webhook о review-событии.
6. Сервис обновляет внутреннее состояние PR.
7. Клиент может запросить текущее состояние PR через API сервиса.

## 6. Границы MVP

### Входит в MVP

- REST API для создания PR-задачи.
- Интеграция с GitHub REST API.
- Назначение одного reviewer'а.
- Webhook endpoint для приема review events.
- Хранение состояния PR в БД.
- Обновление статуса PR после review event.
- Swagger или хотя бы ручная API-документация.
- Dockerfile и `docker-compose.yml`.
- Unit-тесты на сервисный слой.

### Разумные ограничения MVP

- Один repository provider: GitHub.
- Один reviewer на запрос.
- Считаем, что ветки уже существуют.
- Не создаем ветки автоматически.
- Не делаем merge из сервиса.
- Не обрабатываем сложные политики branch protection глубже базового отказа от GitHub API.

## 7. Нефункциональные требования

Проект должен показывать базовую production-minded разработку:

- таймауты на HTTP-клиент;
- structured logging;
- идемпотентность webhook-обработки хотя бы на базовом уровне;
- конфигурация через env;
- graceful shutdown;
- health endpoint;
- понятные HTTP status codes;
- отсутствие секретов в коде.

## 8. Предлагаемая архитектура

Оптимальная структура для junior+/strong junior:

```text
/cmd/api
/internal/config
/internal/handler/http
/internal/service
/internal/client/github
/internal/repository/postgres
/internal/domain
/internal/logger
/internal/webhook
/migrations
/docs
```

### Слои

- `handler` принимает HTTP и маппит request/response.
- `service` содержит бизнес-логику.
- `client/github` инкапсулирует вызовы GitHub API.
- `repository` работает с БД.
- `domain` хранит сущности и статусы.

Это уже хорошая, но не перегруженная архитектура.

## 9. API, которое стоит сделать

### `POST /api/v1/pull-requests`

Создает PR и назначает reviewer'а.

Пример запроса:

```json
{
  "owner": "acme",
  "repo": "backend-service",
  "title": "Add payment retry logic",
  "body": "This PR adds retry logic for payment requests",
  "head": "feature/payment-retry",
  "base": "main",
  "reviewer": "john-dev"
}
```

Пример ответа:

```json
{
  "id": 1,
  "provider": "github",
  "repo": "acme/backend-service",
  "pr_number": 42,
  "status": "review_pending",
  "reviewer": "john-dev"
}
```

### `GET /api/v1/pull-requests/{id}`

Возвращает текущее состояние PR внутри сервиса.

### `POST /api/v1/webhooks/github`

Принимает webhook от GitHub по review events.

### `GET /healthz`

Проверка, что сервис жив.

## 10. Модель данных

Минимально нужна таблица `pull_requests`.

Поля:

- `id`
- `provider`
- `owner`
- `repo`
- `pr_number`
- `title`
- `head_branch`
- `base_branch`
- `reviewer`
- `status`
- `created_at`
- `updated_at`

Статусы можно зафиксировать так:

- `created`
- `review_pending`
- `approved`
- `review_rejected`

Если хочется аккуратнее, можно хранить еще `last_review_state` и `provider_payload_id` для идемпотентности webhook'ов.

## 11. Бизнес-логика

### Create PR flow

1. Провалидировать входной запрос.
2. Сохранить запись со статусом `created` или создать ее после вызова GitHub API.
3. Создать PR через GitHub API.
4. Назначить reviewer'а.
5. Сохранить `pr_number` и статус `review_pending`.

### Review webhook flow

1. Проверить подпись webhook.
2. Проверить тип события.
3. Найти PR во внутренней БД.
4. Если review state = `approved`, перевести PR в `approved`.
5. Если review state = `changes_requested` или аналогичный отказ, перевести PR в `review_rejected`.
6. Сохранить обновленное состояние.

## 12. Технологический стек

Рекомендуемый стек:

- Go 1.24+ или актуальная стабильная версия;
- `chi` или `gin`, лучше `chi`, если хочется показать более явную архитектуру;
- `pgx` для PostgreSQL;
- `log/slog` для логирования;
- `testify` для тестов;
- `golang-migrate` или SQL migrations вручную.

Если хочется минимализма, можно вообще обойтись стандартной библиотекой HTTP, но для junior проекта `chi` выглядит практичнее.

## 13. Что будет считаться strong junior уровнем

Проект должен показывать не количество технологий, а качество решений.

### Признаки хорошего уровня

- код разбит на логичные пакеты;
- нет giant handler'ов с бизнес-логикой;
- ошибки обрабатываются, а не теряются;
- внешние зависимости спрятаны за интерфейсами там, где это оправдано;
- есть тесты на ключевые сценарии;
- можно локально поднять проект за 1-2 команды;
- README объясняет запуск и сценарий работы;
- есть понятные ограничения системы.

### Что ухудшит впечатление

- чрезмерная "чистая архитектура" ради архитектуры;
- 15 интерфейсов на 3 структуры;
- отсутствие БД при наличии stateful логики;
- webhook без проверки подписи;
- отсутствие тестов и примеров запросов.

## 14. Что можно сделать как plus, но не обязательно

Если MVP готов, дальше можно добавить:

- автоматический merge после approve;
- поддержку нескольких reviewer'ов;
- retry для временных ошибок GitHub API;
- worker для асинхронного merge;
- outbox/inbox паттерн;
- OpenAPI/Swagger;
- Prometheus metrics;
- GitLab adapter как второй provider.

Но это именно второй этап, не первый.

## 15. Этапы реализации

### Этап 1. Каркас

- инициализация проекта;
- конфиг;
- роутер;
- healthcheck;
- Docker Compose с PostgreSQL.

### Этап 2. Домен и БД

- схема таблицы;
- модели;
- repository слой;
- базовые тесты repository или service.

### Этап 3. GitHub integration

- client для GitHub API;
- создание PR;
- назначение reviewer'а;
- обработка ошибок GitHub.

### Этап 4. Основной API

- endpoint создания PR;
- endpoint получения PR по id;
- валидация request/response.

### Этап 5. Webhook

- прием webhook;
- проверка подписи;
- смена статуса после review event.

### Этап 6. Полировка

- логирование;
- тесты;
- README;
- примеры `.env`;
- Postman/curl examples.

## 16. Критерии готовности

Проект можно считать завершенным, если:

1. Локально поднимаются API и PostgreSQL.
2. По HTTP можно создать сущность PR automation request.
3. Сервис реально создает PR в GitHub.
4. Сервис реально назначает reviewer'а.
5. Webhook от GitHub меняет статус во внутренней БД.
6. Есть тесты на ключевую бизнес-логику.
7. Есть README с инструкцией запуска.

## 17. Итоговое решение

Если фиксировать цель строго, то я бы зафиксировал ее так:

**Делаем один Go-микросервис с интеграцией только в GitHub, PostgreSQL для хранения состояния, REST API для создания pull request, назначение reviewer'а и webhook endpoint для review events с обновлением статуса.**

Это хороший, реалистичный и демонстрабельный strong junior проект.

## 18. Следующее практическое решение

После этого документа правильный следующий шаг:

1. Утвердить MVP именно в таком scope.
2. На его основе написать `README.md` с кратким описанием проекта.
3. Затем создать технический backlog: сущности, endpoints, migration, service methods, GitHub client methods, tests.
