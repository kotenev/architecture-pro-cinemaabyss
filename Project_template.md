## Изучены [README.md](README.md) файл и структура проекта.

## Задание 1

1. To be архитектура КиноБездны в виде контейнерной диаграммы в нотации С4:
[ссылка на файл](https://github.com/kotenev/architecture-pro-cinemaabyss/blob/cinema/diagrams/container/CinemaAbyss_Container.puml)

## Задание 2

### 1. Proxy
Реализован бесшовный переход с применением паттерна Strangler Fig в части реализации прокси-сервиса (API Gateway), с помощью которого можно постепенно переключать траффик, используя фиче-флаг.

Реализован сервис на Go в ./src/microservices/proxy.

### 2. Kafka
Создан MVP сервис events, который при вызове API создаёт и сам же читает сообщения в топике Kafka.

    - Разработан сервис на Go с consumer'ами и producer'ами.
    - Реализован API, при вызове которого создаются события User/Payment/Movie и обрабатываются внутри сервиса с записью в лог
    - Новый сервис добавлен в docker-compose
 
Приложены скриншот тестов и скриншот состояния топиков Kafka http://localhost:8090 :
- [ссылка на скриншот тестов](https://github.com/kotenev/architecture-pro-cinemaabyss/blob/cinema/docs/screenshots/CinemaAbyss%20Postman%20all%20tests%20passed%20screenshot.png)
- [ссылка на скриншот состояния топиков Kafka](https://github.com/kotenev/architecture-pro-cinemaabyss/blob/cinema/docs/screenshots/CinemaAbyss%20kafka%20topics%20screenshot.png)

## Задание 3

Команда начала переезд в Kubernetes для лучшего масштабирования и повышения надежности. 
Вам, как архитектору осталось самое сложное:
 - реализовать CI/CD для сборки прокси сервиса
 - реализовать необходимые конфигурационные файлы для переключения трафика.

### CI/CD

 В папке .github/worflows доработан деплой новых сервисов proxy и events в docker-build-push.yml , чтобы api-tests при сборке отрабатывали корректно при отправке коммита в новую ветку.

Теперь сборка отрабатывает и в github registry появляются образы.

### Proxy в Kubernetes

Приложены скриншот вывода при вызове https://cinemaabyss.example.com/api/movies и скриншот вывода event-service после вызова тестов:
- [ссылка на скриншот вывода при вызове https://cinemaabyss.example.com/api/movies](https://github.com/kotenev/architecture-pro-cinemaabyss/blob/cinema/docs/screenshots/CinemaAbyss%20api%20movies%20URL%20browser%20screenshot.png)
- [ссылка на скриншот вывода events-service после вызова тестов](https://github.com/kotenev/architecture-pro-cinemaabyss/blob/cinema/docs/screenshots/CinemaAbyss%20kubernetes%20events-service%20logs%20screenshot.png)

## Задание 4
Приложены скриншот развертывания helm и скриншот вывода https://cinemaabyss.example.com/api/movies :
- [ссылка на скриншот развёртывания helm](https://github.com/kotenev/architecture-pro-cinemaabyss/blob/cinema/docs/screenshots/CinemaAbyss%20helm%20deployment%20screenshot.png)
- [ссылка на скриншот вывода https://cinemaabyss.example.com/api/movies](https://github.com/kotenev/architecture-pro-cinemaabyss/blob/cinema/docs/screenshots/CinemaAbyss%20api%20movies%20URL%20browser%20screenshot%20(after%20helm%20deployment).png)

# Задание 5
Приложены скриншоты работы circuit breaker'а:
- [ссылка на первый скриншот работы circuit breaker'а](https://github.com/kotenev/architecture-pro-cinemaabyss/blob/cinema/docs/screenshots/CinemaAbyss%20circuit%20breaker%20result%20%20screenshot%201.png)
- [ссылка на второй скриншот работы circuit breaker'а](https://github.com/kotenev/architecture-pro-cinemaabyss/blob/cinema/docs/screenshots/CinemaAbyss%20circuit%20breaker%20result%20%20screenshot%202.png)
