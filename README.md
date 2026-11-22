# дз 3 docker

## запуск

```sh
cp .env.example .env
docker compose up -d --build
```

переменные в `.env`: `APP_PORT` (по умолчанию 8080) и `GREETING` (по умолчанию `Hello, Docker!`)
в `compose.yaml` сервисы `app` (`STORAGE=redis`, `REDIS_URL=redis://redis:6379/0`, порт `${APP_PORT}:8000`) и `redis` (`--appendonly yes`, volume `redis_data:/data`, порт наружу не проброшен), у обоих настроен healthcheck

остановить: `docker compose down`

## проверки

### 4 балла

```sh
docker build -t counter .
docker run -d --name counter -p 8080:8000 -e GREETING="Hi, Docker!" counter
curl http://127.0.0.1:8080/
curl -X POST http://127.0.0.1:8080/increment
curl -X POST http://127.0.0.1:8080/increment
docker logs counter
```

```
{"greeting": "Hi, Docker!", "count": 0, "hostname": "7834ad0c2aa4", "storage": "sqlite"}
{"greeting": "Hi, Docker!", "count": 1, "hostname": "7834ad0c2aa4", "storage": "sqlite"}
{"greeting": "Hi, Docker!", "count": 2, "hostname": "7834ad0c2aa4", "storage": "sqlite"}

2026-11-22 20:39:33,680 listening port=8000 storage=sqlite
2026-11-22 20:39:34,682 http 172.17.0.1 "GET / HTTP/1.1" 200 -
2026-11-22 20:39:34,697 http 172.17.0.1 "POST /increment HTTP/1.1" 200 -
2026-11-22 20:39:34,708 http 172.17.0.1 "POST /increment HTTP/1.1" 200 -
```

### 6 и 8 баллов

если в `.env` прописать `APP_PORT=8085` и `GREETING=Hello from Compose!`

запуск и 3 инкремента:

```sh
docker compose up -d --build
docker compose ps
curl http://127.0.0.1:8085/
curl -X POST http://127.0.0.1:8085/increment
curl -X POST http://127.0.0.1:8085/increment
curl -X POST http://127.0.0.1:8085/increment
```

```
hw3-app-1     hw3-app            Up 5 seconds (healthy)    0.0.0.0:8085->8000/tcp
hw3-redis-1   redis:7.4-alpine   Up 11 seconds (healthy)   6379/tcp

{"greeting": "Hello from Compose!", "count": 0, "hostname": "0440331a9864", "storage": "redis"}
{"greeting": "Hello from Compose!", "count": 1, "hostname": "0440331a9864", "storage": "redis"}
{"greeting": "Hello from Compose!", "count": 2, "hostname": "0440331a9864", "storage": "redis"}
{"greeting": "Hello from Compose!", "count": 3, "hostname": "0440331a9864", "storage": "redis"}
```

пересоздать контейнеры (`down` без `-v` и `up -d`):

```sh
docker compose down
docker compose up -d
curl http://127.0.0.1:8085/
```

```
{"greeting": "Hello from Compose!", "count": 3, "hostname": "4ec28ebe7123", "storage": "redis"}
```

остановить redis:

```sh
docker compose stop redis
curl -i http://127.0.0.1:8085/healthz
curl -i http://127.0.0.1:8085/readyz
curl -i -X POST http://127.0.0.1:8085/increment
```

```
GET /healthz: 200 {"status": "alive"}
GET /readyz: 503 {"error": "storage_unavailable"}
POST /increment: 503 {"error": "storage_unavailable"}
```

запустить redis (без перезапуска app), дождаться готовности:

```sh
docker compose start redis
curl http://127.0.0.1:8085/readyz
curl http://127.0.0.1:8085/
curl -X POST http://127.0.0.1:8085/increment
```

```
{"status": "ready", "storage": "redis"}
{"greeting": "Hello from Compose!", "count": 3, "hostname": "4ec28ebe7123", "storage": "redis"}
{"greeting": "Hello from Compose!", "count": 4, "hostname": "4ec28ebe7123", "storage": "redis"}
```
