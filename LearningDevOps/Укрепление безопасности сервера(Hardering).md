
## Отключаем авторизацию по паролю и остальные плюшки
Для того чтобы обезопасить наш сервер нам нужно отключить авторизацию по SSH. Для этого нам нужно зайти в конфиг данной командой:
```terminal
sudo nano /etc/ssh/sshd_config
```
[[Нужные команды для DevOps Engineer#Nano|nano]]
Далее, мы ищем следующие параметры(если строки закомментированы символом `#`, убери его): `PasswordAuthentication no`, `PubkeyAuthentication yes`, `PermitRootLogin no`

Подробнее:
  ◦ `PasswordAuthentication no` — это самое важное. Запрещает вход по паролю.

    ◦ `PubkeyAuthentication yes` — разрешает вход по ключам.

    ◦ `PermitRootLogin no` — запрещает заходить под `root` напрямую. Это заставит тебя заходить под обычным пользователем и использовать `sudo`.

Для быстрого поиска рекомендую использовать CTRL+W 

После того как мы все подправили, мы сохраняем конфиг и проверяем данной командой что все окей:
```terminal
sudo sshd -t
```
Если ничего не вывело после того как мы ее отпрвили. значит все в порядке

Теперь нам нужно перезапустить службу SSH:
```terminal
sudo systemctl restart ssh
```
[[Нужные команды для DevOps Engineer#Systemctl|systemctl]]
## Настройка сетевого экрана(FireWall UFW)

Прежде чем мы перейдем к настройки сетевого экрана, проверим работу сервиса:
```terminal
sudo ufw status
```
[[Нужные команды для DevOps Engineer#UFW(Сетевой экран/FireWall)|UFW]]
Мы увидили что сервис не активен:
```treminal
Status: inactive
maki@lablearning:~$ 
```

прежде чем его активировать, разрешим доступ к порту SHH( порт 22), иначе доступ к серверу потеряем:
```terminal
sudo ufw allow ssh
#или
sudo ufw allow 22/tcp
```
[[Нужные команды для DevOps Engineer#UFW(Сетевой экран/FireWall)|UFW]]
После этого мы можем активировать файервол:
```termianl
sudo ufw enable
```
[[Нужные команды для DevOps Engineer#UFW(Сетевой экран/FireWall)|UFW]]
Соглашаемся с тем что мы можем потерять подключение:
```terminal
Command may disrupt existing ssh connections. Proceed with operation (y|n)? y
Firewall is active and enabled on system startup 
```

и заключительная команда для проверки что все применилось успешно:
```terminal
sudo ufw status verbose
```
[[Нужные команды для DevOps Engineer#UFW(Сетевой экран/FireWall)|UFW]]
Должен быть такой ответ:
```terminal
Status: active
Logging: on (low)
Default: deny (incoming), allow (outgoing), disabled (routed)
New profiles: skip

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW IN    Anywhere                  
22/tcp (v6)                ALLOW IN    Anywhere (v6)             

maki@lablearning:~$ 
```
Теперь все порты, кроме 22-го, закрыты. Если в будущем мы поставим веб-сервер Nginx, нам нужно будет отдельно открыть порты 80 (HTTP) и 443 (HTTPS)

## Управление привилегиями (Sudo и группы)
Для добавления пользователя в группу администраторов нужно использовать данную команду:
```terminal
sudo usermod -aG sudo имя_пользователя
```
[[Нужные команды для DevOps Engineer#UserMod|usermod]]
Далее проверяем права у пользователя:
```terminal
id имя_пользователя
```
вот что у меня выдало:
```terminal
uid=1000(maki) gid=1000(maki) groups=1000(maki),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),101(lxd)
```
Для понимания есть таблица: [[Таблица прав доступа Linux]]

## Обновление системы
Команды которые нужно обязательно выполнять при каждой авторизации на сервер(чтобы держать в актуальном режиме сервер):
```terminal
sudo apt update
sudo apt upgrade -y
```
[[Нужные команды для DevOps Engineer#Advanced Package Tool|apt]]
**Команды для регулярного использования:**

• `sudo apt update` — обновить список доступных пакетов.

• `sudo apt upgrade -y` — установить все доступные обновления безопасности.

## Анализ логов авторизации
Если сервер выведен в WAN сеть, можно проверить сколько было авторизацией данной командой:
```terminal
sudo tail -f /var/log/auth.log
```
 [[Нужные команды для DevOps Engineer#Tail|tail]]
 Пример auth.log:
 ```terminal
 2026-02-25T15:41:32.991729+00:00 lablearning systemd-logind[1070]: New session 22 of user maki.
2026-02-25T15:41:37.119366+00:00 lablearning sshd[3158]: Received disconnect from 192.168.101.201 port 56649:11: disconnected by user
2026-02-25T15:41:37.119615+00:00 lablearning sshd[3158]: Disconnected from user maki 192.168.101.201 port 56649
2026-02-25T15:41:37.120517+00:00 lablearning sshd[3102]: pam_unix(sshd:session): session closed for user maki
2026-02-25T15:41:37.125198+00:00 lablearning systemd-logind[1070]: Session 22 logged out. Waiting for processes to exit.
2026-02-25T15:41:37.126433+00:00 lablearning systemd-logind[1070]: Removed session 22.
2026-02-25T15:45:01.831824+00:00 lablearning CRON[3169]: pam_unix(cron:session): session opened for user root(uid=0) by root(uid=0)
2026-02-25T15:45:01.835555+00:00 lablearning CRON[3169]: pam_unix(cron:session): session closed for user root
2026-02-25T15:45:40.594406+00:00 lablearning sudo:     maki : TTY=pts/2 ; PWD=/home/maki ; USER=root ; COMMAND=/usr/bin/tail -f /var/log/auth.log
2026-02-25T15:45:40.597652+00:00 lablearning sudo: pam_unix(sudo:session): session opened for user root(uid=0) by maki(uid=1000)
 ```
 