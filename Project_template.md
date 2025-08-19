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
Компания планирует активно развиваться и для повышения надежности, безопасности, реализации сетевых паттернов типа Circuit Breaker и канареечного деплоя вам как архитектору необходимо развернуть istio и настроить circuit breaker для monolith и movies сервисов.

```bash

helm repo add istio https://istio-release.storage.googleapis.com/charts
helm repo update

helm install istio-base istio/base -n istio-system --set defaultRevision=default --create-namespace
helm install istio-ingressgateway istio/gateway -n istio-system
helm install istiod istio/istiod -n istio-system --wait

helm install cinemaabyss .\src\kubernetes\helm --namespace cinemaabyss --create-namespace

kubectl label namespace cinemaabyss istio-injection=enabled --overwrite

kubectl get namespace -L istio-injection

kubectl apply -f .\src\kubernetes\circuit-breaker-config.yaml -n cinemaabyss

```

Тестирование

# fortio
```bash
kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.25/samples/httpbin/sample-client/fortio-deploy.yaml -n cinemaabyss
```

# Get the fortio pod name
```bash
FORTIO_POD=$(kubectl get pod -n cinemaabyss | grep fortio | awk '{print $1}')

kubectl exec -n cinemaabyss $FORTIO_POD -c fortio -- fortio load -c 50 -qps 0 -n 500 -loglevel Warning http://movies-service:8081/api/movies
```
Например,

```bash
kubectl exec -n cinemaabyss fortio-deploy-b6757cbbb-7c9qg  -c fortio -- fortio load -c 50 -qps 0 -n 500 -loglevel Warning http://movies-service:8081/api/movies
```

Вывод будет типа такого

```bash
IP addresses distribution:
10.106.113.46:8081: 421
Code 200 : 79 (15.8 %)
Code 500 : 22 (4.4 %)
Code 503 : 399 (79.8 %)
```
Можно еще проверить статистику

```bash
kubectl exec -n cinemaabyss fortio-deploy-b6757cbbb-7c9qg -c istio-proxy -- pilot-agent request GET stats | grep movies-service | grep pending
```

И там смотрим 

```bash
cluster.outbound|8081||movies-service.cinemaabyss.svc.cluster.local;.upstream_rq_pending_total: 311 - столько раз срабатывал circuit breaker
You can see 21 for the upstream_rq_pending_overflow value which means 21 calls so far have been flagged for circuit breaking.
```

Приложите скриншот работы circuit breaker'а

Удаляем все
```bash
istioctl uninstall --purge
kubectl delete namespace istio-system
kubectl delete all --all -n cinemaabyss
kubectl delete namespace cinemaabyss
```
