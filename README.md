## Laboratory work VIII

Данная лабораторная работа посвещена изучению систем автоматизации развёртывания и управления приложениями на примере **Docker**

```sh
$ open https://docs.docker.com/get-started/
```

## Tasks

- [x] 1. Создать публичный репозиторий с названием **lab08** на сервисе **GitHub**
- [x] 2. Ознакомиться со ссылками учебного материала
- [x] 3. Выполнить инструкцию учебного материала
- [x] 4. Составить отчет и отправить ссылку личным сообщением в **Slack**

## Tutorial

```sh
$ export GITHUB_USERNAME=<имя_пользователя>
```

```
$ cd ${GITHUB_USERNAME}/workspace
$ pushd .
$ source scripts/activate
```
Клонируем репозиторий с лр7 в лр8
```sh
$ git clone https://github.com/${GITHUB_USERNAME}/lab07 lab08
$ cd lab08
$ git submodule update --init
$ git remote remove origin
$ git remote add origin https://github.com/${GITHUB_USERNAME}/lab08
```
Создаем докер файл и задаем образ 
```sh
$ cat > Dockerfile <<EOF
FROM ubuntu:18.04
EOF
```
Устанавливаем компиляторы и cmake 
```sh
$ cat >> Dockerfile <<EOF

RUN apt update
RUN apt install -yy gcc g++ cmake
EOF
```
Копируем файлы в директорию print и делаем её активной с помощью WORDIR
```sh
$ cat >> Dockerfile <<EOF

COPY . print/
WORKDIR print
EOF
```
Собираем 
```sh
$ cat >> Dockerfile <<EOF

RUN cmake -H. -B_build -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=_install
RUN cmake --build _build
RUN cmake --build _build --target install
EOF
```
Устанавливаем постоянные переменные среды 
```sh
$ cat >> Dockerfile <<EOF

ENV LOG_PATH /home/logs/log.txt
EOF
```
Указываем постоянное хранилище 
```sh
$ cat >> Dockerfile <<EOF

VOLUME /home/logs
EOF
```
Делаем активной следующую директорию
```sh
$ cat >> Dockerfile <<EOF

WORKDIR _install/bin
EOF
```
Запускаем исполняемый файл
```sh
$ cat >> Dockerfile <<EOF

ENTRYPOINT ./demo
EOF
```
Собираем образ
```sh
$ docker build -t logger .
```
Просматриваем доступные образы
```sh
$ docker images
```
Создаем директорию logs
run - создание и запуск контейнера.
Флаг -i — это сокращение для --interactive. Благодаря этому флагу поток STDIN поддерживается в открытом состоянии даже если контейнер к STDIN не подключён.
Флаг -t — это сокращение для --tty. Благодаря этому флагу выделяется псевдотерминал, который соединяет используемый терминал с потоками STDIN и STDOUT контейнера.
Флаг -v указывает постоянное хранилище
```sh
$ mkdir logs
$ docker run -it -v "$(pwd)/logs/:/home/logs/" logger
text1
text2
text3
<C-D>
```
Просматриваем состояние образа
```sh
$ docker inspect logger
```
Выводим содержимое log.txt для проверки
```sh
$ cat logs/log.txt
```

```sh
$ gsed -i 's/lab07/lab08/g' README.md
```
Создадим .yml файл для создания образа
```sh
$ vim .travis.yml
/lang<CR>o
services:
- docker<ESC>
jVGdo
script:
- docker build -t logger .<ESC>
:wq
```
Сохраняем и пушим все изменения
```sh
$ git add Dockerfile
$ git add .travis.yml
$ git commit -m"adding Dockerfile"
$ git push origin master
```
Устанавливаем автоматический вход в GitHub
```sh
$ travis login --auto
$ travis enable
```
