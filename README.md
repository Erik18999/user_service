# User Service

## Описание

Микросервис для управления пользователями в веб-приложении **CorporationX** — социальной сети для стартаперов, IT-специалистов и обычных пользователей. Отвечает за профили пользователей, менторство, цели (goals), навыки (skills), организацию/участие в событиях (events), премиум-доступ, подписки на других пользователей, рекомендации и загрузку аватаров.

## Реализованные фичи

### Менторство: получение и удаление менти/менторов
Пользователь может выступать в роли ментора и вести менти. Реализованы операции получения списка менти/менторов и удаления связи менторства в обе стороны.

- [`MentorshipController`](src/main/java/school/faang/user_service/controller/mentorship/MentorshipController.java)
- [`MentorshipServiceImpl`](src/main/java/school/faang/user_service/service/impl/MentorshipServiceImpl.java)

**Технологии:** Spring Web, Spring Data JPA, MapStruct, Lombok, JUnit 5 + Mockito

## Стек

- Java 17
- Spring Boot 3
- Spring Data JPA
- PostgreSQL
- Redis (Pub/Sub)
- MinIO
- Feign Client
- MapStruct
- Liquibase
- JUnit 5, Mockito

## CI/CD

Настроен GitHub Actions пайплайн для проверки Pull Request'ов в ветку `werewolf-master-stream8`: сборка проекта, прогон тестов, автоматический комментарий в PR при падении сборки.

- [`.github/workflows/ci.yml`](.github/workflows/ci.yml)

## Запуск

1. Поднять инфраструктуру:
```bash
git clone https://github.com/Erik18999/infra.git
cd infra
./run.sh
```
2. Запустить сервис (порт 8080)

## Swagger UI

http://localhost:8080/swagger-ui/index.html
