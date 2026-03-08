Здесь представлено руководство по настройке CachyOS после установки.

<details>

<summary>Оглавление</summary>

- [Установщик AUR](#установщик-aur)
- [Установка Google Chrome](#установка-google-chrome)
- [Установка docker и docker-compose](#установка-docker-и-docker-compose)
  - [Донастройка](#донастройка)
    - [Добавление в группу docker](#добавление-в-группу-docker)
    - [Включение docker демона при запуске системы](#включение-docker-демона-при-запуске-системы)
    - [Лимит логов для docker контейнеров](#лимит-логов-для-docker-контейнеров)
- [Auto login (KDE)](#auto-login-kde)
  - [Без шифрования](#без-шифрования)
    - [Настройка](#настройка)
  - [С шифрованием](#с-шифрованием)
    - [Настройка](#настройка-1)

</details>

# Установщик AUR
AUR - пользовательский репозиторий ARCH (дистрибутив на котором основан CachyOS). В нем пользователи могу публиковать свои собственные приложения, которых нет в официальном репозитории.

> [!warning]
> Не стоит качать оттуда всё подряд, репозиторий из-за того что общедоступный содержит вирусные приложения, поэтому в том что вы качаете, нужно быть предельно аккуратным.

Открыть консоль и выполнить
```shell
sudo pacman -S yay
```

# Установка Google Chrome
Google Chrome браузер - думаю что-то дополнительно говорить излишне.

**Предварительные требования**
- Установщик AUR (paru или yay)

```shell
yay -S google-chrome
```

# Установка docker и docker-compose
Docker - средство для управления контейнерами (изолированными процессами и ресурсами) (практически как ВМ, но без эмуляции "железа"). Так как в официальной документации нет информации по установке в arch-based дистрибутивы, то весь наиболее используемый набор ставится этой командой:
```shell
sudo pacman -S --needed docker docker-buildx docker-compose
```

## Донастройка
### Добавление в группу docker
```shell
sudo groupadd docker || true
# Вообще конечно выдавать себе права docker это не лучшая идея, потому что они эквивалентны root. Но без них пользоваться docker будет та еще каторга с постоянным вводом sudo и без автодополнения
sudo usermod -aG docker $USER
```

### Включение docker демона при запуске системы
Проверка что демон (фоновый процесс) включился
```shell
sudo systemctl enable --now docker
systemctl status docker
```

### Лимит логов для docker контейнеров
Ограничить лимит логов, чтобы логи контейнеров не сожрали всё доступное место
```shell
sudo mkdir -p /etc/docker
sudo nano /etc/docker/daemon.json
```
Добавить в файл следующее содержимое
```json
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "5"
  }
}
```

Перезапустить демон докера
```shell
sudo systemctl daemon-reload && sudo systemctl restart docker
```

Проверить что докер работает
```shell
docker run -it --rm hello-world
```

Должно быть выведено что-то такое
```
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
17eec7bbc9d7: Pull complete 
ea52d2000f90: Download complete 
Digest: sha256:ef54e839ef541993b4e87f25e752f7cf4238fa55f017957c2eb44077083d7a6a
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```

# Auto login (KDE)

<details>

<summary>Tags</summary>

`autologin`, `auto login`, `plasma autologin`, `sddm autologin`, `kde autologin`, `cachyos autologin`, `pam_autologin`, `pam autologin`, `kwallet autologin`, `kwallet-pam`, `kdewallet autologin`

</details>

Для включения autologin есть несколько способов.
1. Без шифрования паролей
2. С сохранением широфвания паролей

**Контекст**
При установке новых версий CachyOS с окружением KDE используется kde-wallet (kwallet), где хранятся ваши пароли (chrome, discord, steam и т.д.). При включении компьютера и авторизации под вашей ученой записью пароль также передается в kwallet и приложения в автозапуске запускаются без проблем.

## Без шифрования
Вы можете либо вообще отключить kwallet, либо просто задать ему пустой пароль. В обоих случаях ваши пароли от других приложений будут лежать как есть, без какого-либо шифрования.

**Плюсы**
- Просто настраивается

**Минусы**
- Пароли все еще хранятся в вашем домашнем каталоге, а значит вирус запущенный от вашей учетной записи с легкостью может получить к ним доступ.

### Настройка
1. Открыть `KwalletManager` (можно написать в поиске программ)
2. Настройка -> Сменить пароль -> Ничего не вводить -> Применить -> Согласиться на сообщение, что пароль не безопасен.
3. Открыть настройки KDE
   - Параметры системы -> `Login Screen`
   - Включить `Automatically log in`.
   - Нажать `Применить` справа внизу
   - Нажать `Apply Plasma Settings` справа вверху

4. Перезагрузиться
5. Запроса на ввод пароля НЕ должно быть. Также НЕ должен был высветиться запрос от kdewallet manager на ввод пароля. Приложения из автозагрузки успешно запустились.


## С шифрованием
В этом варианте kwallet все еще имеет свой пароль и шифрует ваши пароли, но пароль от kwallet лежит в директории доступной только для root пользователя и во время включения системы передается в kwallet.

**Плюсы**
- Пароли все еще хранятся в шифрованном состоянии

**Минусы**
- Сложность настройки


### Настройка

**Предварительные требования**
- Установщик AUR (paru или yay)

0. Сделайте снапшот или бекап

1. Установить зависимости
```shell
sudo pacman -S --needed kwallet-pam
yay -S pam_autologin
```

Дальше идут разделы под разное окружение. Выберите нужное вам.

<details>

<summary>(Вариант 1) Настройка для KDE Plasma 6 с Plasma Login Manager</summary>

1. Подготовка файлов
```shell
# Скопировать файл plasmalogin-autologin. Нужно чтобы при обновлении пакетов ничего не перезатерлось
sudo cp /usr/lib/pam.d/plasmalogin-autologin /etc/pam.d/plasmalogin-autologin

# Создать файл, где будет лежать пароль для pam_autologin и выдать корректные права
sudo touch /etc/security/autologin.conf
sudo chmod 600 /etc/security/autologin.conf
```

2. Отредактировать файл `/etc/pam.d/plasmalogin-autologin`

Нужно будет добавить 1 строку и изменить 2 строки
```shell
# Добавлена
auth        optional    pam_autologin.so # Эта строка обязана быть выше, чем следующие две
# Изменены
# -auth       optional    pam_kwallet5.so
auth        optional    pam_kwallet5.so
# -session    optional    pam_kwallet5.so auto_start
session     optional    pam_kwallet5.so auto_start kwalletd=/usr/bin/ksecretd
```

Итоговый файл должен выглядеть как-то так
```conf
#%PAM-1.0

# SPDX-License-Identifier: CC0-1.0
# SPDX-FileCopyrightText: none

auth        required    pam_env.so
auth        required    pam_faillock.so preauth
auth        required    pam_shells.so
auth        required    pam_nologin.so
auth        optional    pam_autologin.so
auth        optional    pam_kwallet5.so
auth        required    pam_permit.so

account     include     system-local-login
password    include     system-local-login

session     include     system-local-login
session     optional    pam_kwallet5.so auto_start kwalletd=/usr/bin/ksecretd
```

</details>


<details>

<summary>(Вариант 2) для KDE Plasma 6 с SDDM</summary>

1. Подготовка файлов
```shell
# Создать файл, где будет лежать пароль для pam_autologin и выдать корректные права
sudo touch /etc/security/autologin.conf
sudo chmod 600 /etc/security/autologin.conf
```

1. Отредактировать файл `/etc/pam.d/sddm-autologin`

Нужно будет добавить 1 строку и изменить 2 строки
```shell
# Добавлена
auth        optional    pam_autologin.so # Эта строка обязана быть выше, чем следующие две
# Изменены
# -auth       optional    pam_kwallet5.so
auth        optional    pam_kwallet5.so
# -session    optional    pam_kwallet5.so auto_start
session     optional    pam_kwallet5.so auto_start kwalletd=/usr/bin/ksecretd
```

Итоговый файл должен выглядеть как-то так
```conf
#%PAM-1.0
auth        required    pam_env.so
auth        required    pam_faillock.so preauth
auth        required    pam_shells.so
auth        required    pam_nologin.so
auth        optional    pam_autologin.so 
auth        required    pam_permit.so
-auth       optional    pam_gnome_keyring.so
auth        optional    pam_kwallet5.so
account     include     system-local-login
password    include     system-local-login
session     include     system-local-login
-session    optional    pam_gnome_keyring.so auto_start
session     optional    pam_kwallet5.so auto_start kwalletd=/usr/bin/ksecretd

```

</details>

<details>

<summary>(Вариант 3) для KDE Plasma 5 с SDDM</summary>

1. Подготовка файлов
```shell
# Создать файл, где будет лежать пароль для pam_autologin и выдать корректные права
sudo touch /etc/security/autologin.conf
sudo chmod 600 /etc/security/autologin.conf
```

1. Отредактировать файл `/etc/pam.d/sddm-autologin`

Нужно будет добавить 1 строку и изменить 2 строки
```shell
# Добавлена
auth        optional    pam_autologin.so # Эта строка обязана быть выше, чем следующие две
# Изменены
# -auth       optional    pam_kwallet5.so
auth        optional    pam_kwallet5.so
# -session    optional    pam_kwallet5.so auto_start
session     optional    pam_kwallet5.so auto_start
```

Итоговый файл должен выглядеть как-то так
```conf
#%PAM-1.0
auth        required    pam_env.so
auth        required    pam_faillock.so preauth
auth        required    pam_shells.so
auth        required    pam_nologin.so
auth        optional    pam_autologin.so 
auth        required    pam_permit.so
-auth       optional    pam_gnome_keyring.so
auth        optional    pam_kwallet5.so
account     include     system-local-login
password    include     system-local-login
session     include     system-local-login
-session    optional    pam_gnome_keyring.so auto_start
session     optional    pam_kwallet5.so auto_start

```

</details>


3. Открыть настройки KDE
   - Параметры системы -> `Login Screen`
   - Убедиться что `Automatically log in` **ВЫКЛЮЧЕН**. Мы его включим позже. Сейчас он должен быть выключен.

4. Перезагрузиться
5. Ввести пароль к учетной записи
6. Проверить что пароль записался в `/etc/security/autologin.conf`
```shell
sudo cat /etc/security/autologin.conf
# Будут выведены "кракозябры" по типу `�Hl�H�d���`
# Если так, то все успешно. Если файл пустой, значит что-то пошло не так. Удостоверьтесь в правильности выполненных действий и окружения.
```

7. Открыть настройки KDE
   - Параметры системы -> `Login Screen`
   - Включить `Automatically log in`.
   - Нажать `Применить` справа внизу
   - Нажать `Apply Plasma Settings` справа вверху

8. Перезагрузиться
9. Запроса на ввод пароля НЕ должно быть. Также НЕ должен был высветиться запрос от kdewallet manager на ввод пароля. Приложения из автозагрузки успешно запустились.


