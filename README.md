Здесь представлено руководство по настройке CachyOS после установки.

<details>

<summary>Оглавление</summary>

- [Auto login (KDE)](#auto-login-kde)
  - [Без шифрования](#без-шифрования)
    - [Настройка](#настройка)
  - [С шифрованием](#с-шифрованием)
    - [Настройка](#настройка-1)

</details>

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

Предварительные требования
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


