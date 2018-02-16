# DevOps Homework-17 by Eugeny Kobushka

## Оглавление документации домашних работ

1. [Домашняя работа № 14](docs/hw-14.md)
2. [Домашняя работа № 15](docs/hw-15.md)
3. [Домашняя работа № 16](docs/hw-16.md)
4. [Домашняя работа № 17](docs/hw-17.md)

## Готовим стенд для выполнения домашней работы

```bash
docker-machine create --driver google --google-project docker-193604 --google-zone europe-west1-b  --google-machine-type g1-small --google-machine-image $(gcloud compute images list --filter ubuntu-1604-lts --uri) docker-host
docker-machine ls
eval "$(docker-machine env docker-host)"
```

## Разбираемся с работой сети в Docker (none, host, bridge)

### None network driver

```bash
# none
docker run --network none --rm -d --name net_test joffotron/docker-net-tools -c "sleep 100"
docker exec -ti net_test ifconfig
docker exec -ti net_test ping -c 3 127.0.0.1
```

**Вывод:** Работает только loopback. Пинг этого интерефейса работает. Других интерфейсов в контейнере, как мы видим не существует.

### Host network driver

```bash
# host
docker run --network host --rm -d --name net_test joffotron/dockernet-tools -c "sleep 100"
docker exec -ti net_test ifconfig
docker-machine ssh docker-host ifconfig
```

**Ответ на вопрос:** Вывод этих команд не отличается друг от друга и показывает одинаковую информацию.

```bash
# host - запускаем 4 раза
docker run --network host -d nginx
docker ps
```

**Ответ на вопрос:** Что мы видим в выводе команды ps? <br>
Запущен только один контейнер.

```bash
docker ps -a
# берем ID контейнера чтобы глянуть в его логи

docker logs e3e59fc067b0
# в логах видим такие строки

2018/02/15 19:09:57 [emerg] 1#1: bind() to 0.0.0.0:80 failed (98: Address already in use)
nginx: [emerg] bind() to 0.0.0.0:80 failed (98: Address already in use)
```

**Ответ на вопрос:** Как думаете почему? <br>
Контейнеры не могут запуститься, потому что порт 80 уже используется и не может быть использован для последующих контейнеров.

### Docker-network (*)

```bash
# создаем симлинк
docker-machine ssh docker-host  sudo ln -s /var/run/docker/netns /var/run/netns

# смотрим net-namespace для драйвера none
docker run --network none --rm -d --name net_test joffotron/docker-net-tools -c "sleep 100"
docker-machine ssh docker-host sudo ip netns
08b0402d2129

# смотрим net-namespace для драйвера host
docker run --network host --rm -d --name net_test joffotron/dockernet-tools -c "sleep 100"
docker-machine ssh docker-host sudo ip netns
default

# Примечание на память
# ip netns exec <namespace> <command>
# позволит выполнять команды в выбранном namespace
```

**Вывод:** <br>
Если создается контейнер с драйвером **none**, то namespace создается. Если контейнер запускается в
с драйвером **host**, то не создается.

### Bridge network driver

```bash
# создаем docker-сети
docker network create back_net --subnet=10.0.2.0/24
docker network create front_net --subnet=10.0.1.0/24

# Запустим контейнеры
docker run -d --network=front_net -p 9292:9292 --name ui ekobushka/ui:1.0
docker run -d --network=back_net --name comment ekobushka/comment:1.0
docker run -d --network=back_net --name post ekobushka/post:1.0
docker run -d --network=back_net --name mongo_db --network-alias=post_db --network-alias=comment_db mongo:latest

# Подключим контейнеры ко второй сети
docker network connect front_net post
docker network connect front_net comment
```

### Изучаем структуру сетевого стека с помощью bridge-utils

Список команд для изучения сетевого стека

```bash
# смотрим ID всех сетей
docker network ls

# смотрим bridge-интерфейсы
ifconfig | grep br
brctl show br-<ID_interface>

# изучаем iptables
sudo iptables -nL -t nat

# смотрим docker-proxy (должен быть хотя бы один процесс)
ps ax | grep docker-proxy
```

## docker-compose

Для чистоты выполнения пересоздадим docker-host

```bash
docker-machine rm docker-host
# создаем новый чистый хост
docker-machine create --driver google --google-project docker-193604 --google-zone europe-west1-b  --google-machine-type g1-small --google-machine-image $(gcloud compute images list --filter ubuntu-1604-lts --uri) docker-host
eval "$(docker-machine env docker-host)"
```

Создаем для ознакомления файл docker-compose.yml и проверяем работу docek-compose

```bash
# Обновил версию docker-compose до 1.19.0 ругался на версию 3.3 yml-файла
sudo rm -rf /usr/local/bin/docker-compose && sudo curl -L https://github.com/docker/compose/releases/download/1.19.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose && sudo chmod +x /usr/local/bin/docker-compose && docker-compose --version
# либо устанавливаем через pip
# pip install docker-compose

# собираем контейнеры с компонентами нашего приложения docker-compose
docker-compose up -d

# смотрим что запущено с помощью docker-compose
docker-compose ps

            Name                          Command             State           Ports
--------------------------------------------------------------------------------------------
redditmicroservices_comment_1   puma                          Up
redditmicroservices_post_1      python3 post_app.py           Up
redditmicroservices_post_db_1   docker-entrypoint.sh mongod   Up      27017/tcp
redditmicroservices_ui_1        puma                          Up      0.0.0.0:9292->9292/tcp

# смотрим ip чтобы проверить что все работает корректно и правильно
docker-machine ls

# После проверки правильности работы останавливаем все контейнеры
docker-compose stop
# или docker-compose down
# Если надо отключить volume то добавляем ключ
# docker-compose down --volumes
```

После того как мы убедились что наше приложение корректно и правильно работает, добавим параметризацию для наших параметров и запишем их в файл **.env**
Перезапустим docker-compose и убедимся что все работает корректно.

**Ответ на вопрос:** <br>
Узнайте как образуется базовое имя проекта. Можно ли его задать? Если можно то как?

Имя проекта задается в файле **.env** переменной окружения `COMPOSE_PROJECT_NAME` или

```bash
docker-compose -p COMPOSE_PROJECT_NAME up -d
# может понадобиться например для запуска нескольких копий нашего проекта
# с разными именами они будут запущены...
```

Внесем изменения в файл .env и пересоздадим контейнеры чтобы убедиться что имя проекта изменилось

```bash
docker-compose ps

      Name                   Command             State           Ports
-------------------------------------------------------------------------------
reddit_comment_1   puma                          Up
reddit_post_1      python3 post_app.py           Up
reddit_post_db_1   docker-entrypoint.sh mongod   Up      27017/tcp
reddit_ui_1        puma                          Up      0.0.0.0:9292->9292/tcp
```

## Docker-compose override
Есть перевод от ОТУС <https://habrahabr.ru/company/otus/blog/337688/>
автор: Tully (статья от 11 сентября 2017)

Статья помогла разобраться. Автору перевода спасибо.

### Выполнение

* Изменять код каждого из приложений, не выполняя сборку образа (подключим volume)
* Запускать puma для руби приложений в дебаг режиме с двумя воркерами (флаги --debug и -w 2)

Добавлен файл docker-compose.override.yml

```bash
docker-compose -f docker-compose.yml -f docker-compose.override.yml up -d
```
