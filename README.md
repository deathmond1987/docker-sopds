изменения:
- Смена версии alpine 3.16 -> 3.20
- Так как в новых версиях alpine предоставляется несколько версий postgresql закрепил в Dockerfile пакет postgresql14
- Добавлен supervisord для передачи сигнала выключения в postgresql (раньше контейнер выключался по таймауту а не по gracefull shutdown)
- Добавлена генерация конфигов supervisord в /scripts/start.sh ( костыль но рабочий)
- Если включена опция SOPDS_TMBOT_ENABLE=true то генерится конфиг сервиса telegram для supervisord
- Если указан токен в опции SOPDS_TELEBOT_API_TOKEN то токен передается в приложение при старте контейнера (опция SOPDS_TMBOT_ENABLE=true должна быть включена)
- Добавлен скрипт ожидания включения postgresql ко всем сервисам. Чтобы при старте успевшие запуститься сервисы не кидали ошибку подключения
- Добавлен nginx
- Добавлен docker-compose.yml
- Переменные вынес в .env файл в docker-compose.yml
- Для дебага временно вынес в /dev/stdout все логи которые нашел
- Добавлена переменная SOPDS_TELEGRAM_USER_NAME. Если у переменной указать имя вашего аккаунта telegram - у него появится доступ к этому инстансу sopds 
---------------------------------------------------------------
https://github.com/ichbinkirgiz/sopds


# Introduction

Dockerfile to build a Simple OPDS server docker image.
http://www.sopds.ru

# Installation

Pull the latest version of the image from the docker.

```
docker pull ghcr.io/zveronline/sopds
```

Alternately you can build the image yourself.

```
docker build -t ghcr.io/zveronline/sopds https://github.com/zveronline/docker-sopds.git
```

# Quick Start

Run the image

```
docker run --name sopds -d \
   --volume /path/to/library:/library:ro \
   --publish 8001:8001 \
   ghcr.io/zveronline/sopds
```

This will start the sopds server and you should now be able to browse the content on port 8081.

```
docker run --name sopds -d \
   --volume /path/to/library:/library:ro \
   --volume /path/to/database:/var/lib/pgsql \
   --publish 8001:8001 \
   ghcr.io/zveronline/sopds
```

Also you can store postgresql database on external storage.

```
docker run --name sopds -d \
   --volume /path/to/library:/library:ro \
   --env 'DB_USER=sopds' \
   --env 'DB_NAME=sopds' \
   --env 'DB_PASS=sopds' \
   --env 'DB_HOST=""' \
   --env 'DB_PORT=""' \
   --env 'EXT_DB=True' \
   --publish 8001:8001 \
   ghcr.io/zveronline/sopds
```


# Create superuser

By default the superuser will be created with predefined name "admin" and password "admin". But you can manage it via appropriate environmental variables:
```bash
docker run --name sopds -d \
   --volume /path/to/library:/library:ro \
   --volume /path/to/database:/var/lib/pgsql \
   --env 'SOPDS_SU_NAME="your_name_for_superuser"' \
   --env 'SOPDS_SU_EMAIL='"your_mail_for_superuser@your_domain"' \
   --env 'SOPDS_SU_PASS="your_password_for_superuser"' \
   --publish 8001:8001 \
   ghcr.io/zveronline/sopds
```

# Scan library

```bash
docker exec -ti sopds bash
python3 manage.py sopds_util setconf SOPDS_SCAN_START_DIRECTLY True
```

# Telegram bot autostart

To do this you need to use a variable:
SOPDS_TMBOT_ENABLE=True
