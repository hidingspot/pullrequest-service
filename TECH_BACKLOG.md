# Technical Backlog

## Goal

Собрать `MVP` Go-микросервиса для GitHub pull request automation:

- создать pull request через GitHub API;
- назначить reviewer'а;
- сохранить состояние в PostgreSQL;
- принять webhook review event;
- обновить внутренний статус PR;
- отдать текущее состояние через HTTP API.

## Milestone 1. Bootstrap

- [ ] Инициализировать Go-модуль.
- [ ] Добавить базовую структуру директорий.
- [ ] Подключить HTTP router.
- [ ] Реализовать `GET /healthz`.
- [ ] Добавить конфиг через env.
- [ ] Настроить structured logging.

## Milestone 2. Infrastructure

- [ ] Добавить `Dockerfile`.
- [ ] Добавить `docker-compose.yml` c PostgreSQL.
- [ ] Подготовить `.env.example`.
- [ ] Добавить SQL migration для таблицы `pull_requests`.
- [ ] Подключить PostgreSQL из приложения.

## Milestone 3. Domain Model

- [ ] Описать сущность `PullRequest`.
- [ ] Зафиксировать статусы: `created`, `review_pending`, `approved`, `review_rejected`.
- [ ] Описать DTO для create request/response.
- [ ] Описать payload для GitHub review webhook.

## Milestone 4. Repository Layer

- [ ] Реализовать repository для `pull_requests`.
- [ ] Добавить методы:
- [ ] `Create`
- [ ] `GetByID`
- [ ] `GetByProviderPRNumber`
- [ ] `UpdateStatus`
- [ ] Покрыть repository happy-path тестами или отложить это в пользу service tests.

## Milestone 5. GitHub Client

- [ ] Реализовать HTTP client для GitHub API.
- [ ] Добавить метод `CreatePullRequest`.
- [ ] Добавить метод `RequestReviewers`.
- [ ] Обработать основные ошибки GitHub API.
- [ ] Настроить auth через GitHub token.
- [ ] Добавить timeout и базовый retry не делать в MVP.

## Milestone 6. Service Layer

- [ ] Реализовать use case `CreatePullRequest`.
- [ ] Реализовать use case `GetPullRequest`.
- [ ] Реализовать use case `HandleReviewWebhook`.
- [ ] Обеспечить обновление состояния после `approved`.
- [ ] Обеспечить обновление состояния после `changes_requested`.
- [ ] Добавить идемпотентность webhook-обработки на базовом уровне.

## Milestone 7. HTTP Layer

- [ ] Реализовать `POST /api/v1/pull-requests`.
- [ ] Реализовать `GET /api/v1/pull-requests/{id}`.
- [ ] Реализовать `POST /api/v1/webhooks/github`.
- [ ] Добавить валидацию входных данных.
- [ ] Добавить корректные HTTP status codes и error responses.

## Milestone 8. Security and Reliability

- [ ] Проверять подпись GitHub webhook.
- [ ] Не хранить секреты в репозитории.
- [ ] Добавить graceful shutdown.
- [ ] Добавить request logging middleware.

## Milestone 9. Testing

- [ ] Покрыть service layer unit-тестами.
- [ ] Протестировать create PR сценарий через mocks.
- [ ] Протестировать review webhook сценарии:
- [ ] approve
- [ ] changes requested
- [ ] unknown event

## Milestone 10. Documentation

- [ ] Обновить `README.md` после появления реального запуска.
- [ ] Добавить примеры `curl`-запросов.
- [ ] Описать переменные окружения.
- [ ] Описать ограничения MVP.

## First Implementation Order

Рекомендуемый порядок без распыления:

1. Go module + папки + `healthz`
2. Config + logger
3. Docker Compose + PostgreSQL
4. Migration + repository
5. GitHub client
6. Create PR endpoint
7. Webhook endpoint
8. Tests
9. README polishing

## Good Commit Plan

Такую работу лучше коммитить небольшими кусками:

1. `chore: add gitignore and technical backlog`
2. `chore: bootstrap go service structure`
3. `feat: add config and health endpoint`
4. `feat: add postgres repository and migrations`
5. `feat: add github pull request client`
6. `feat: implement pull request creation flow`
7. `feat: handle github review webhooks`
8. `test: cover service layer with unit tests`
9. `docs: update readme with setup and usage`

