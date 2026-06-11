# PR Automation Service

HTTP-микросервис на Go для автоматизации pull request workflow в GitHub.

Сервис принимает HTTP-запрос на создание pull request, создает PR через GitHub API, назначает reviewer'а, принимает webhook-события review и обновляет внутренний статус pull request.

## Why This Project

Это pet-project уровня `strong junior`, цель которого показать:

- проектирование небольшого backend-сервиса на Go;
- интеграцию с внешним API;
- работу с webhook'ами;
- хранение состояния в PostgreSQL;
- аккуратную архитектуру, тесты и документацию.

## MVP Scope

В первую версию входят:

- создание pull request через GitHub API;
- назначение одного reviewer'а;
- сохранение состояния PR в PostgreSQL;
- webhook endpoint для review events;
- обновление статуса PR после review;
- REST API для получения состояния PR;
- Docker Compose для локального запуска.

Не входят в MVP:

- GitLab и другие платформы;
- UI;
- автоматический merge;
- очереди и асинхронные воркеры;
- создание веток и работа с git локально.

## Tech Stack

- Go
- GitHub REST API
- PostgreSQL
- Docker Compose
- `chi`
- `pgx`
- `slog`

## Planned API

### `POST /api/v1/pull-requests`

Создает pull request и назначает reviewer'а.

### `GET /api/v1/pull-requests/{id}`

Возвращает текущее состояние pull request внутри сервиса.

### `POST /api/v1/webhooks/github`

Принимает webhook от GitHub по review events.

### `GET /healthz`

Проверка доступности сервиса.

## Project Status

Сейчас проект находится на этапе проектирования и фиксации MVP.

Ближайшие шаги:

1. Собрать каркас Go-сервиса.
2. Добавить конфигурацию и HTTP-роутинг.
3. Поднять PostgreSQL через Docker Compose.
4. Реализовать создание PR через GitHub API.
5. Добавить webhook-обработку review events.

## Architecture Direction

Планируемая структура проекта:

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
```

## Documentation

- Scope проекта: [PROJECT_SCOPE.md](./PROJECT_SCOPE.md)

## Future Improvements

После MVP можно добавить:

- auto-merge после approve;
- поддержку нескольких reviewer'ов;
- retries для GitHub API;
- metrics;
- OpenAPI/Swagger;
- второго provider'а.

## Repository Goals

Этот репозиторий ведется как демонстрационный инженерный проект, поэтому отдельный фокус:

- чистая структура;
- понятная история коммитов;
- хороший README;
- без секретов в репозитории;
- воспроизводимый локальный запуск.

