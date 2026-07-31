# Zorth Web API

Монолитный backend для компиляции и исполнения программ на языке Zorth. Сервис оборачивает [транслятор и модель стекового процессора](https://github.com/zavar152-cv/stack-processor) в REST API, сохраняет запросы и результаты в PostgreSQL и ограничивает доступ с помощью JWT и ролей.

Это промежуточный этап развития проекта между автономной моделью процессора и [микросервисной версией](https://github.com/zavar152-cv/stack-processor-ms).

## Возможности

- компиляция Zorth в машинный код;
- выполнение машинного кода на тактовой модели процессора;
- единый pipeline «compile + execute»;
- сохранение запросов, листингов, результатов и debug-трасс;
- JWT-аутентификация;
- роли `USER`, `VIP` и `ADMIN`;
- REST API для истории запусков и управления пользователями;
- PostgreSQL и миграции Liquibase;
- интеграционные тесты с Testcontainers и REST Assured;
- контейнеризация приложения и базы данных.

## Архитектура

```text
HTTP request
    ↓
Spring Security / JWT
    ↓
Zorth Controller
    ├── Translator → machine code
    ├── Processor  → output + debug trace
    └── PostgreSQL → requests and results
```

## Стек

Java 17, Spring Boot 3, Spring MVC, Spring Security, Spring Data JPA, PostgreSQL, Liquibase, JWT, Docker Compose, JUnit 5, Testcontainers, REST Assured и JaCoCo.

## Запуск через Docker Compose

Скопируйте пример переменных окружения и замените секреты:

```bash
cp .env.example .env
docker compose up --build
```

API будет доступен по адресу `http://localhost:25565/api/v1`.

Основные маршруты:

- `POST /auth/signIn` — получить JWT;
- `POST /zorth/compile` — скомпилировать программу;
- `POST /zorth/execute` — выполнить машинный код;
- `POST /zorth/pipeline` — выполнить полный pipeline;
- `GET /zorth/getRequests` — получить историю запросов.

## Локальный запуск

Нужны Java 17 и PostgreSQL. Перед запуском задайте как минимум ключ подписи JWT:

```bash
export TOKEN_SIGNING_KEY='replace-with-at-least-32-random-bytes'
export SPRING_DATASOURCE_URL='jdbc:postgresql://localhost:5432/db-highload'
export SPRING_DATASOURCE_USERNAME='user_db'
export SPRING_DATASOURCE_PASSWORD='highload'

./mvnw spring-boot:run
```

## Тесты

```bash
./mvnw test
```

Интеграционные тесты поднимают PostgreSQL через Testcontainers, поэтому им нужен работающий Docker daemon.

## Безопасность

`.env` исключён из Git. Значения из `.env.example` предназначены только как шаблон; для любого доступного извне окружения используйте случайный JWT-ключ и уникальные пароли. Старый ключ, ранее находившийся в истории репозитория, нельзя считать секретным.

## Участники

Backend и архитектура: [Ярослав Абузов](https://github.com/zavar152-cv). В истории репозитория также есть вклад Данилы Пименова.
