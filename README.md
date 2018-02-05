# DevOps Homework-15 by Eugeny Kobushka

## **Задание:** Выполнить последовательность действий из методического пособия

### **Выполнение**

1. Создаем проект docker-ID и переинициализируем GCP для работы с новым проектом
```bash
gcloud init
```
2. Получаем файл аутентификации для docker-machine
```bash
gcloud auth application-default login
```
3. Создаем docker-host и проверяем что все успешно создано
```bash

# Создадим докер-хост
docker-machine create --driver google --google-project docker-193604 --google-zone europe-west1-b  --google-machine-type g1-small --google-machine-image $(gcloud compute images list --filter ubuntu-1604-lts --uri) docker-host

# Проверим его работу
docker-machine ls

``` 
4. Создаем файлы нашего репозитория:
  * Dockerfile
  * mongod.conf
  * db_conf
  * start.sh

5. Сборка образа
```bash

# Собираем образ
docker build -t reddit:latest .

# Проверяем результат
docker images -a

```
6. Запускаем контейнер
```bash

docker run --name reddit -d --network=host reddit:latest

# Убедимся что без нужного правила файрвола не работает - открываем полученный IP в браузере

```
7. Разрешаем входящий TCP-трафик на порт 9292 выполнив команду:
```bash

gcloud compute firewall-rules create reddit-app  --allow tcp:9292 --priority=65534  --target-tags=docker-machine  --description="Allow TCP connections"  --direction=INGRESS

# проверяем наш сервис - все открываемся

```
9. Docker-hub
* Проходим регистрацию (если нет)
* Загрузим наш образ на docker hub
```bash

# регистрация
docker login

# Push image
docker tag reddit:latest ekobushka/otus-reddit:1.0
docker push ekobushka/otus-reddit:1.0

```

<br><br>

# DevOps Homework-14 by Eugeny Kobushka (выполнено)

<details><br>

## **Задание:** Выполнить все шаги по методичке...
<details><br>
### 1. Устанавливаем docker на Ubuntu Linux:

Описание установки есть на страничке:<br>

https://docs.docker.com/engine/installation/linux/docker-ce/ubuntu/


Привожу выполненные команды по установке докера

```bash
# remove old version
sudo apt-get remove docker docker-engine docker.io

# install new version
sudo apt-get install apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo apt-key fingerprint 0EBFCD88
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
sudo apt-get update
sudo apt-get install docker-ce

# check working docker
sudo docker run hello-world

# install docker-compose
sudo curl -L https://github.com/docker/compose/releases/download/1.18.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
docker-compose --version


# install docker-machine
curl -L https://github.com/docker/machine/releases/download/v0.13.0/docker-machine-`uname -s`-`uname -m` >/tmp/docker-machine && chmod +x /tmp/docker-machine && sudo cp /tmp/docker-machine /usr/local/bin/docker-machine
docker-machine version
```
</details>

### 2. Проверяем что каждый раз создается новый контейнер
<details>

```bash
docker run -it ubuntu:16.04 /bin/bash
root@95580ac35bf0:/# echo "Hello world!" > /tmp/hello.txt
root@95580ac35bf0:/# exit
exit

docker run -it ubuntu:16.04 /bin/bash
root@91105c4abc99:/# cat /tmp/hello.txt
cat: /tmp/hello.txt: No such file or directory
root@91105c4abc99:/# exit
exit
```
</details>

### 3. Cмотрим ранее созданные контейнеры
<details>

```bash
docker ps -a --format "table {{.ID}}\t{{.Image}}\t{{.CreatedAt}}\t{{.Names}}"
CONTAINER ID        IMAGE               CREATED AT                      NAMES
91105c4abc99        ubuntu:16.04        2018-01-28 20:24:58 +0300 MSK   heuristic_poitras
95580ac35bf0        ubuntu:16.04        2018-01-28 20:24:02 +0300 MSK   boring_ardinghelli
cf69e430aae6        hello-world         2018-01-28 20:11:41 +0300 MSK   laughing_liskov
```
</details>

### 4. Запуск существующего контейнера и подсключение к нему
<details>
```bash
sudo docker start 95580ac35bf0
sudo docker attach 95580ac35bf0
root@95580ac35bf0:/# cat /tmp/hello.txt 
Hello world!
</details>

# оставим контейнер работать и выйдем из него
<details>

Ctrl+p, Ctrl+q

sudo docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
95580ac35bf0        ubuntu:16.04        "/bin/bash"         8 minutes ago       Up About a minute                       boring_ardinghelli

</details>

# Подключаемся к существующему контейнеру
<details>

sudo docker start 95580ac35bf0
sudo docker attach 95580ac35bf0
<ENTER>
root@95580ac35bf0:/
```

### 5. Создадим образ на основе работающего контейнера
```bash
sudo docker commit 95580ac35bf0 ekobushka/ubuntu-tmp-file 
sha256:073267cbd155f3d79db2ee6e492b52dfca41434efe512e0fe5128c979e2da89d

sudo docker images
REPOSITORY                  TAG                 IMAGE ID            CREATED             SIZE
ekobushka/ubuntu-tmp-file   latest              073267cbd155        19 seconds ago      112MB
ubuntu                      16.04               0458a4468cbc        2 days ago          112MB
elasticsearch               2                   705cb8c5336c        2 months ago        574MB
mongo                       3                   d22888af0ce0        2 months ago        361MB
hello-world                 latest              48b5124b2768        12 months ago       1.84kB
graylog2/server             2.1.1-1             58a2b309b960        16 months ago       619MB
```
</details>

### 6. После всего этого остановим работающий контейнер, командой
<details>
```bash
sudo docker kill $(sudo docker ps -q)
95580ac35bf0
```
</details>

### 7. Просмотрим занятое место образами, контейнерами и разделами
<details>

```bash
sudo docker system df
TYPE                TOTAL               ACTIVE              SIZE                RECLAIMABLE
Images              6                   3                   1.666GB             1.666GB (99%)
Containers          4                   0                   166B                166B (100%)
Local Volumes       2                   0                   32.71kB             32.71kB (100%)
Build Cache                                                 0B                  0B
```
### 8. Удалим все неиспользуемые и не запущенные контейнеры
<details>

```bash
sudo docker rm $(sudo docker ps -a -q)
288db9f165a6
91105c4abc99
95580ac35bf0
cf69e430aae6
```
</details>

### 9. Удалим все образы от которых не зависят запущенные контейнеры
<details>

```bash
sudo docker rmi $(sudo docker images -q)     
Untagged: ekobushka/ubuntu-tmp-file:latest
Deleted: sha256:073267cbd155f3d79db2ee6e492b52dfca41434efe512e0fe5128c979e2da89d
Deleted: sha256:423b8711bcfb27da51b65f8936c889c40fd8f8f19ee3af8b2d6fae6268b6c58d
Untagged: ubuntu:16.04
Untagged: ubuntu@sha256:e27e9d7f7f28d67aa9e2d7540bdc2b33254b452ee8e60f388875e5b7d9b2b696
Deleted: sha256:0458a4468cbceea0c304de953305b059803f67693bad463dcbe7cce2c91ba670
Deleted: sha256:77e6ddba346d8ad1e436256f6373dede5af4002006981b7d4116c561c759cefa
Deleted: sha256:8db758ab2fdb54da0aec53aeac876934337e6170f5a8c8872b3d4171e3d465b7
Deleted: sha256:a7fc6b405fe8ef71edfa6163d1dc9f1cb1df426049eefaa7d388e9df21a061ad
Deleted: sha256:5a3e35538f7f2e2727c8ac92f08c30002b9e8a77737de0dab91244344d59f69b
Deleted: sha256:ff986b10a018b48074e6d3a68b39aad8ccc002cdad912d4148c0f92b3729323e

<весь вывод не привожу... он длинный, т.к. в списке были контейнеры не по заданию>

```
</details>

### В лог файл **docker-1.log** добавлены выводы для задания со *
