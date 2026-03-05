---
Tags:
  - " "
  - "#linux"
  - "#systemd"
  - "#admin"
  - "#devops"
---
## Определение
**Systemd** — это менеджер системы и сервисов в Linux. Он заменяет старые системы инициализации (Init) и управляет процессами в виде "юнитов".

### Основные команды systemctl
- `start / stop / restart` — управление состоянием здесь и сейчас.
- `enable / disable` — управление автозагрузкой при старте ОС.
- `status` — просмотр состояния, PID и последних строк логов.

sudo systemctl status ssh       # Проверить, запущен ли сервис
sudo systemctl stop ssh         # Остановить (осторожно, не делай этого в удаленной сессии!)
sudo systemctl start ssh        # Запустить
sudo systemctl enable ssh       # Добавить в автозагрузку (чтобы стартовал при включении сервера)
sudo journalctl -u ssh -n 20    # Посмотреть последние 20 строк логов конкретно для SSH 

## Задания
Проверка статуса у сервиса UFW. Выполняем данную команду:
```terminal
systemctl status ufw
```

Мы получаем такой ответ:
```terminal
maki@lablearning:~$ sudo systemctl status ufw
● ufw.service - Uncomplicated firewall
     Loaded: loaded (/usr/lib/systemd/system/ufw.service; enabled; preset: enabled)
     Active: active (exited) since Wed 2026-03-04 10:02:15 UTC; 1 day 9h ago
       Docs: man:ufw(8)
    Process: 694 ExecStart=/usr/lib/ufw/ufw-init start quiet (code=exited, status=0/SUCCESS)
   Main PID: 694 (code=exited, status=0/SUCCESS)
        CPU: 92ms

Mar 04 10:02:14 lablearning systemd[1]: Starting ufw.service - Uncomplicated firewall...
Mar 04 10:02:15 lablearning systemd[1]: Finished ufw.service - Uncomplicated firewall.
maki@lablearning:~$ 
```
Видим что он выключен и на данный момент активен

Смотрим упавшие сервисы:
```terminal
systemctl --failed
```

И такой у нас вывод:
```terminal
maki@lablearning:~$ systemctl --failed
  UNIT LOAD ACTIVE SUB DESCRIPTION

0 loaded units listed.
maki@lablearning:~$ 
```

При помози данной команды мы можем смотреть логи в реальном времени:
```terminal
sudo journalctl -f
```

Пример:
```terminal
maki@lablearning:~$ sudo journalctl -f
Mar 05 19:43:47 lablearning sudo[5213]: pam_unix(sudo:session): session opened for user root(uid=0) by maki(uid=1000)
Mar 05 19:43:47 lablearning sudo[5213]: pam_unix(sudo:session): session closed for user root
Mar 05 19:44:50 lablearning sudo[5218]:     maki : TTY=pts/1 ; PWD=/home/maki ; USER=root ; COMMAND=/usr/bin/journalctl -f
Mar 05 19:44:50 lablearning sudo[5218]: pam_unix(sudo:session): session opened for user root(uid=0) by maki(uid=1000)
Mar 05 19:44:52 lablearning sudo[5218]: pam_unix(sudo:session): session closed for user root
Mar 05 19:45:01 lablearning CRON[5223]: pam_unix(cron:session): session opened for user root(uid=0) by root(uid=0)
Mar 05 19:45:01 lablearning CRON[5224]: (root) CMD (command -v debian-sa1 > /dev/null && debian-sa1 1 1)
Mar 05 19:45:01 lablearning CRON[5223]: pam_unix(cron:session): session closed for user root
Mar 05 19:47:16 lablearning sudo[5226]:     maki : TTY=pts/1 ; PWD=/home/maki ; USER=root ; COMMAND=/usr/bin/journalctl -f
Mar 05 19:47:16 lablearning sudo[5226]: pam_unix(sudo:session): session opened for user root(uid=0) by maki(uid=1000)

```

Далее, мы посмотрим на аналог диспетчера задач в Linux. Вот пару команд:
```terminal
systemd-cgtop
```

Вот так она выглядит:
```terminal
CGroup                                                                                                                                Tasks   %CPU   Memory  Input/s Output/s
/                                                                                                                                       252    2.0   387.1M        -        -
user.slice                                                                                                                               12    1.3    18.4M        -        -
user.slice/user-1000.slice                                                                                                               12    1.3    18.3M        -        -
user.slice/user-1000.slice/session-244.scope                                                                                              4    1.3     4.5M        -        -
system.slice                                                                                                                             55    0.1   710.5M        -        -
system.slice/open-vm-tools.service                                                                                                        4    0.1     5.4M        -        -
system.slice/multipathd.service                                                                                                           7    0.0    22.8M        -        -
system.slice/udisks2.service                                                                                                              6    0.0     7.2M        -        -
system.slice/fwupd.service                                                                                                                6    0.0    31.6M        -        -
dev-hugepages.mount                                                                                                                       -      -   120.0K        -        -
dev-mqueue.mount                                                                                                                          -      -     4.0K        -        -
init.scope                                                                                                                                1      -     5.0M        -        -
proc-sys-fs-binfmt_misc.mount                                                                                                             -      -     8.0K        -        -
sys-fs-fuse-connections.mount                                                                                                             -      -     4.0K        -        -
sys-kernel-config.mount                                                                                                                   -      -     4.0K        -        -
sys-kernel-debug.mount                                                                                                                    -      -     4.0K        -        -
sys-kernel-tracing.mount                                                                                                                  -      -     4.0K        -        -
system.slice/ModemManager.service                                                                                                         4      -     8.2M        -        -
system.slice/boot.mount                                                                                                                   -      -   108.0K        -        -
system.slice/cron.service                                                                                                                 1      -   484.0K        -        -
system.slice/dbus.service                                                                                                                 1      -     2.2M        -        -
system.slice/nginx.service                                                                                                                3      -     3.7M        -        -
system.slice/polkit.service                                                                                                               4      -     5.5M        -        -
system.slice/rsyslog.service                                                                                                              4      -     3.9M        -        -
system.slice/ssh.service                                                                                                                  1      -     4.3M        -        -
system.slice/ssh.socket                                                                                                                   -      -     8.0K        -        -
system.slice/swap.img.swap                                                                                                                -      -   300.0K        -        -
system.slice/system-getty.slice                                                                                                           -      -    12.9M        -        -
system.slice/system-getty.slice/getty@tty1.service                                                                                        -      -    12.9M        -        -
system.slice/system-modprobe.slice                                                                                                        -      -   116.0K        -        -
system.slice/system-systemd\x2dfsck.slice                                                                                                 -      -   944.0K        -        -
system.slice/systemd-journald.service                                                                                                     1      -    30.3M        -        -
system.slice/systemd-logind.service                                                                                                       1      -     1.8M        -        -
system.slice/systemd-networkd.service                                                                                                     1      -     3.3M        -        -
system.slice/systemd-resolved.service                                                                                                     1      -     7.7M        -        -
```

и вторая команда:
```terminal
htop
```

вот так она выглядит, есть интерактивный CLI:
![[Pasted image 20260305225020.png]]

Понимание между enable и start.

> 1. Enable - добавляет в автозапуск сервис
> 2.  Start - запускает на данный момент сервис

Пример проверки:

```terminal
maki@lablearning:~$ sudo systemctl is-enabled ssh
enabled
maki@lablearning:~$ sudo systemctl is-enabled ufw
enabled
```
### Контрольные вопросы

- 1.В чем разница между командами `systemctl stop` и `systemctl disable`?
-1. Stop у нас отвечает за остановку сервиса в данный момент, т.е. он запустить при следующем запуске системы. Disable убирает программу из автозапуска.
- 2.Какая команда перечитывает конфигурационные файлы юнитов с диска, если ты внес в них изменения?
-2. daemon-reload
- 3.Как посмотреть логи сервиса только за последний час через `journalctl`?
  


### Чек-лист
- [ ] Умею управлять SSH и Firewall через systemctl.
- [ ] Понимаю, как проверить, почему сервис не запустился.
- [ ] Знаю, где искать файлы юнитов (`/etc/systemd/system/`).


**Связанные заметки**: [[SSH Access]], [[Networking Basics]], [[Установка и траблшутинг SSH]], [[Передача SSH ключа на Linux сервер и авторизация по ключу]], [[Создание SSH ключа на Windows]], [[Укрепление безопасности сервера(Hardering)]], [[Изучение Linux]]