# Django Site

Докеризированный сайт на Django для экспериментов с Kubernetes.

Внутри папки k8s_deploy_files содержатся файлы для запуска приложения Django  в инфраструктуре K8S. Код самого приложения содержится в [репозитории](https://github.com/trader-daniil/django_admin)


## Переменные окружения

Для начала нужно будет создать файл с переменными окружения для проекта, используя концепцию [Secrets](https://kubernetes.io/docs/concepts/configuration/secret/)

В файле должны содержаться такие переменные окружения как:

- `secret_key` - ключ от вашего приложения Django.
- `debug` - дебаг-режим. Поставьте `False`.
- `allowed_hosts` - доступные Ip адреса вашего сервера
- `database_url`  - настройка для подлкючения к БД

После создайте Secret командой 

``` shell
kubectl apply -f <Ваш файл>
```

## Deployment

После того, как Вы создали Secret нужно создать [деплоймент](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/). Мы будем загружать проект из [Dockerhub](https://hub.docker.com/repository/docker/traderdaniil/k8s-djangoapp/general)

Создайте Deployment командой

``` shell
kubectl apply -f dj-dep.yaml
```

## Service

Создайте объект [Service](https://kubernetes.io/docs/concepts/services-networking/service/), выполнив команду

```shell
kubectl apply -f dj-service.yaml
```


## Ingress

Если Вы хотите, чтобы к Вашему приложению можно было обратиться с помощью Доменного имени, нужен [Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/). Согласно файлу ingress-example Ваше приложение будет работать на доменном имене star-burger.test. Вам нужно будет добавить это имя в файл [hosts](https://help.reg.ru/support/dns-servery-i-nastroyka-zony/rabota-s-dns-serverami/fayl-hosts-gde-nakhoditsya-i-kak-yego-izmenit) рядом с вашим minikube ip

Создайте его командой

``` shell
kubectl apply -f example-ingress.yaml
```

## Postgres

мы будем поднимать нашу БД при помощи утилиты [helm](https://helm.sh/ru/)

Для начала создайте файл с настройками для БД вида:

``` yaml
architecture: "replication"

auth:
  user: "user"
  postgresPassword: "newPostgresPassword123"
  password: "newUserPassword123"
  replicationPassword: "newReplicationPassword123"

passwordUpdateJob:
  enabled: true
```

После установите БД через helm командой

``` shell
helm install my-pg oci://registry-1.docker.io/bitnamicharts/postgresql -f <Ваш файл>
```

## Удаление сессий

Необходимо автоматически удалять сессии, чтобы осуществить периодическое удаление сессий реализован [CronJob](https://kubernetes.io/ru/docs/concepts/workloads/controllers/cron-jobs/)

Создайте его командой

``` shell
kubectl apply -f sessioncronjob.yaml
```

## Миграции

Для осуществления миграций не обязательно заходить в Pod с приложением и прописывать команды. Достатчно запустить объект [Job](https://kubernetes.io/docs/concepts/workloads/controllers/job/), которых их выполнит


Запустите миграции командой

``` shell
kubectl apply -f migratejob.yaml
```
