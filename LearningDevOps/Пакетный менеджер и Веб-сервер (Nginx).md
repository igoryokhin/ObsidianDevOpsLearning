## Работа с пакетным менеджером APT(Advanced Package Tool) 

- `sudo apt update` — не обновляет программы! Она только скачивает свежий список доступных пакетов из репозиториев.
- `sudo apt upgrade` — а вот эта команда уже устанавливает новые версии того, что у тебя уже стоит.
- `sudo apt install <имя>` — установка.
- `sudo apt purge <имя>` — удаление программы вместе со всеми её конфигами (полезно, если ты «накосячил» в настройках и хочешь начать с нуля).
[[Нужные команды для DevOps Engineer#Advanced Package Tool| apt]]


## Установка и запуск Nginx

Установка Nginx при помощи apt:
```terminal
sudo apt install nginx -y
```
[[Нужные команды для DevOps Engineer#Advanced Package Tool| apt]]
Для понимания, после установки Nginx запускается как сервис(демон) которым управляет systemctl. После установки мы можем проверить жив ли он:

```terminal
sudo systemctl status nginx
```
[[Нужные команды для DevOps Engineer#Systemctl|systemctl]]
вот что мы должны получить:

```termianl
maki@lablearning:~$ sudo systemctl status nginx
● nginx.service - A high performance web server and a reverse proxy server
     Loaded: loaded (/usr/lib/systemd/system/nginx.service; enabled; preset: enabled)
     Active: active (running) since Sat 2026-02-28 12:24:25 UTC; 2min 58s ago
       Docs: man:nginx(8)
    Process: 20222 ExecStartPre=/usr/sbin/nginx -t -q -g daemon on; master_process on; (code=exited, status=0/SUCCESS)
    Process: 20224 ExecStart=/usr/sbin/nginx -g daemon on; master_process on; (code=exited, status=0/SUCCESS)
   Main PID: 20253 (nginx)
      Tasks: 3 (limit: 2267)
     Memory: 2.4M (peak: 5.3M)
        CPU: 33ms
     CGroup: /system.slice/nginx.service
             ├─20253 "nginx: master process /usr/sbin/nginx -g daemon on; master_process on;"
             ├─20256 "nginx: worker process"
             └─20257 "nginx: worker process"

Feb 28 12:24:25 lablearning systemd[1]: Starting nginx.service - A high performance web server and a reverse proxy server...
Feb 28 12:24:25 lablearning systemd[1]: Started nginx.service - A high performance web server and a reverse proxy server.
maki@lablearning:~$ 
```

Далее мы можем добавить данный сервис в автозагрузку данной командой:

```terminal
sudo systemctl enable nginx
```
[[Нужные команды для DevOps Engineer#Systemctl|systemctl]]
вот что мы должны получить в ответ:

```terminal
maki@lablearning:~$ sudo systemctl enable nginx
Synchronizing state of nginx.service with SysV service script with /usr/lib/systemd/systemd-sysv-install.
Executing: /usr/lib/systemd/systemd-sysv-install enable nginx
maki@lablearning:~$ 
```

Далее, для того чтобы увидеть привественную страницу Nginx нам потербуется открыть порты. Это можно сделать несколькими способами:

```terminal
sudo ufw allow 'Nginx Full'
```
[[Нужные команды для DevOps Engineer#UFW(Сетевой экран/FireWall)|UFW]]

Либо отверываем порты HTTP и HTTPS отдельно:

```terminal
sudo ufw allow 80/tcp
```

```terminal
sudo ufw allow 443/tcp
```
[[Нужные команды для DevOps Engineer#UFW(Сетевой экран/FireWall)|UFW]]

80 порт отвечает за HTTP
443 порт отвечает соотвественно за HTTPS

вот что мы должны получить после ввода команд:

```termianl
maki@lablearning:~$ sudo ufw allow 'Nginx Full'
Rule added
Rule added (v6)
maki@lablearning:~$ sudo ufw allow 443/tcp
Rule added
Rule added (v6)
maki@lablearning:~$ sudo ufw allow 80/tcp
Rule added
Rule added (v6)
maki@lablearning:~$ 
```

Далее мы можем залезть в конфиг отображаемой страницы. Она находится по данному пути:

```terminal
/var/www/html/index.nginx-debian.html
```

^6eacfc

После изменения мы получаем данную страницу:

# Welcome to Maki DevOps Lab!

If you see this page, the nginx web server is successfully installed and working. Further configuration is required.

For online documentation and support please refer to [nginx.org](http://nginx.org/).  
Commercial support is available at [nginx.com](http://nginx.com/).

_Thank you for using nginx._


Дополнительно еще можем посмотреть конфигурацию сайта по данному пути:

```terminal
/etc/nginx/sites-available/
```

Там находится файл default который отвечает за работу сайта.

- **/etc/nginx/** — это «мозг» сервера. Здесь лежат инструкции, как Nginx должен себя вести. В Linux каталог `/etc` зарезервирован строго для конфигурационных файлов всей системы.
- **/var/www/** — это «хранилище» данных. В каталоге `/var` лежат файлы, содержимое которых часто меняется в процессе работы (логи, базы данных, веб-страницы).

