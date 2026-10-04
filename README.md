# shared-minio

Отдельный сервис объектного хранилища (MinIO) для бэкендов **chat** и **makeup**.

Раньше MinIO жил в compose-проекте makeup, из-за чего chat тянулся в сеть makeup
и возникали коллизии сетевых алиасов. Теперь это самостоятельный проект:

- поднимает только `minio`;
- публикует порты на localhost (`127.0.0.1:9000` API, `127.0.0.1:9001` console);
- подключается к внешней сети **`shared-minio`**, куда также входят `chat-app`
  и, при необходимости, `makeup-app`; оба обращаются к хранилищу по имени
  `http://minio:9000`;
- данные лежат в нейтральном томе `shared-minio-data`.

## Конфигурация

`.env` (не коммитится, см. `.env.example`):

| Переменная | Назначение |
|-----------|-----------|
| `MINIO_ROOT_USER` | root-логин MinIO |
| `MINIO_ROOT_PASSWORD` | root-пароль MinIO |
| `MINIO_HOST` | домен/IP для `MINIO_DOMAIN` |

Приложения используют те же креды: `MINIO_ACCESS_KEY` = `MINIO_ROOT_USER`,
`MINIO_SECRET_KEY` = `MINIO_ROOT_PASSWORD`.

## Запуск

```bash
docker network create shared-minio   # один раз на хосте
cp .env.example .env                 # и заполнить
docker compose up -d
```

## Автодеплой

При push в `main` GitHub Actions по SSH заходит на VPS, обновляет чекаут в
`DEPLOY_PATH` (`/root/shared-minio`), создаёт сеть `shared-minio` и пересобирает
сервис. Секреты репозитория: `DEPLOY_HOST`, `DEPLOY_USER`, `DEPLOY_SSH_KEY`,
`DEPLOY_PATH`.
