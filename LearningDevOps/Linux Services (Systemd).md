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


### Чек-лист
- [ ] Умею управлять SSH и Firewall через systemctl.
- [ ] Понимаю, как проверить, почему сервис не запустился.
- [ ] Знаю, где искать файлы юнитов (`/etc/systemd/system/`).


**Связанные заметки**: [[SSH Access]], [[Networking Basics]], [[Установка и траблшутинг SSH]], [[Передача SSH ключа на Linux сервер и авторизация по ключу]], [[Создание SSH ключа на Windows]], [[Укрепление безопасности сервера(Hardering)]], [[Изучение Linux]]