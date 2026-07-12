# User Service

## Описание

Микросервис для управления пользователями в веб-приложении **CorporationX** — социальной сети для создателей стартапов, IT-специалистов и обычных пользователей. Отвечает за профили пользователей, менторство, цели (goals), навыки (skills), организацию/участие в событиях (events), премиум-доступ, подписки на других пользователей, рекомендации (для достижения целей), загрузку аватаров пользователей.

## Реализованные фичи

### 1. Уведомление о достижении цели (event-driven, Publisher)
Пользователи приложения CorporationX могут участвовать в различных Проектах. В рамках проекта участники могут назначить себе конкретную цель. При завершении пользователем цели в `user_service` публикуется событие в Redis-топик `goalCompletedChannel`. Событие асинхронно потребляется в `notification_service` (Listener), который отправляет уведомление пользователю на его e-mail. Реализована валидация состояния цели (уже завершена / не назначена пользователю), автоматическое обновление навыков пользователя по завершённой цели, кастомная обработка ошибок публикации.

- [`GoalController`](src/main/java/school/faang/user_service/controller/GoalController.java)
- [`GoalServiceImpl`](src/main/java/school/faang/user_service/service/impl/GoalServiceImpl.java)
- [`GoalCompletedEventPublisher`](src/main/java/school/faang/user_service/publisher/GoalCompletedEventPublisher.java)
- [`RedisConfiguration`](src/main/java/school/faang/user_service/config/RedisConfiguration.java)

**Технологии:** Redis Pub/Sub (Jedis), Spring Data Redis, Jackson (сериализация событий), Lombok, JUnit 5 + Mockito

### 2. Загрузка, получение и удаление аватара пользователя
Пользователь может загрузить картинку для аватара в своём профиле (до 5 МБ), которая автоматически сохраняется в двух версиях — большая (макс. сторона 1080px) и маленькая (макс. сторона 170px). Файлы хранятся в MinIO (S3-совместимое хранилище), в PostgreSQL сохраняются только их идентификаторы. При загрузке файла автоматически происходит валидация входного файла и кастомная обработка ошибок (файл не найден, ошибка обработки изображения, пользователь не найден).

- [`UserAvatarController`](src/main/java/school/faang/user_service/controller/user/UserAvatarController.java)
- [`UserAvatarServiceImpl`](src/main/java/school/faang/user_service/service/impl/UserAvatarServiceImpl.java)
- [`UserAvatarValidator`](src/main/java/school/faang/user_service/validator/userAvatar/UserAvatarValidator.java)
- [`S3Config`](src/main/java/school/faang/user_service/config/minio/S3Config.java)

**Технологии:** Amazon S3 SDK (MinIO), Thumbnailator (сжатие/ресайз изображений), Spring Multipart, Lombok, JUnit 5 + Mockito

### 3. Удаление просроченных премиум-доступов
У пользователей приложения CorporationX есть возможность оформить себе Премиум-подписку. Премиум-подписка пользователя ограничена по времени. По истечении срока данные о премиум-доступе автоматически удаляются из БД. Данная фича реализована через scheduled-задачу с разбиением на батчи и параллельной асинхронной обработкой через отдельный `ThreadPoolTaskExecutor`.

- [`PremiumRemoverScheduler`](src/main/java/school/faang/user_service/scheduler/PremiumRemoverScheduler.java)
- [`PremiumServiceImpl`](src/main/java/school/faang/user_service/service/premium/impl/PremiumServiceImpl.java)
- [`PremiumListPartitioner`](src/main/java/school/faang/user_service/scheduler/premium/PremiumListPartitioner.java)
- [`AsyncConfig`](src/main/java/school/faang/user_service/config/properties/AsyncConfig.java)

**Технологии:** Spring Scheduling (`@Scheduled`, cron), Spring Async (`@Async`, `ThreadPoolTaskExecutor`), `CompletableFuture`, Apache Commons Collections (`ListUtils.partition`), Lombok, JUnit 5 + Mockito

### 4. Менторство: получение и удаление менти/менторов
Пользователь в рамках Проекта, в котором он участвует, может стать Ментором для младших специалистов, либо младший специалист может запросить менторство (наставничество) у более опытного специалиста. Пользователь может выступать в роли ментора и вести менти. Реализованы операции получения списка менти/менторов и удаления связи менторства в обе стороны.

- [`MentorshipController`](src/main/java/school/faang/user_service/controller/mentorship/MentorshipController.java)
- [`MentorshipServiceImpl`](src/main/java/school/faang/user_service/service/impl/MentorshipServiceImpl.java)

**Технологии:** Spring Web, Spring Data JPA, MapStruct, Lombok, JUnit 5 + Mockito

## CI

Настроен GitHub Actions пайплайн для проверки Pull Request'ов в ветку `werewolf-master-stream8`: сборка проекта, прогон тестов, автоматический комментарий в PR при падении сборки.

- [`.github/workflows/ci.yml`](.github/workflows/ci.yml)

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

## Конфигурация

Конфигурация вынесена в типобезопасные `@ConfigurationProperties`-классы ([`S3Properties`](src/main/java/school/faang/user_service/config/properties/S3Properties.java), [`RedisConfigurationProperties`](src/main/java/school/faang/user_service/config/RedisConfigurationProperties.java)), подключаемые через `@EnableConfigurationProperties` в [`UserServiceApplication`](src/main/java/school/faang/user_service/UserServiceApplication.java).

Все настройки сервиса (подключение к БД, Redis, MinIO, Feign-клиенты, cron-выражения и т.д.) собраны в одном файле: [`application.yaml`](src/main/resources/application.yaml).

## Запуск

### Предварительные требования
- Docker и Docker Compose
- JDK 17

### Шаги

1. Поднять инфраструктуру (Postgres, Redis, MinIO, Kafka):
```bash
git clone https://github.com/Erik18999/infra.git
cd infra
./run.sh
```
2. Склонировать и запустить сам сервис (порт 8080):
```bash
git clone https://github.com/Erik18999/user_service.git
cd user_service
```
Открыть проект в IntelliJ IDEA и запустить [`UserServiceApplication`](src/main/java/school/faang/user_service/UserServiceApplication.java).

## Swagger UI

http://localhost:8080/swagger-ui/index.html
