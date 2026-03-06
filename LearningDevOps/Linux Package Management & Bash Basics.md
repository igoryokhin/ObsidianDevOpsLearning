---
tags:
  - " "
  - "#linux"
  - "#apt"
  - "#bash"
  - "#automation"
---
## Ключевые понятия темы

- **Репозиторий**: удаленное хранилище пакетов.
- **APT (Advanced Package Tool)**: стандартный инструмент управления софтом в Debian/Ubuntu.
- **Shebang (****#!****)**: первая строка скрипта, указывающая [интерпретатор](https://ru.wikipedia.org/wiki/%D0%98%D0%BD%D1%82%D0%B5%D1%80%D0%BF%D1%80%D0%B5%D1%82%D0%B0%D1%82%D0%BE%D1%80) (например, `#!/bin/bash`).
- **Переменные**: способ хранения данных в скриптах (например, `NAME="Maki"`)

## Практика

### Обновление и установка
Для того чтобы нам пакет программ, нам понадобиться обновить список из доверенных репозиториев которые уже добавлены в систему. После чего мы сможем обновить приложения:
```terminal
sudo apt update && sudo apt upgrade -y
```

`sudo apt update` -  отвечает за обновление списка из рипозитория
`sudo apt upgrade -y` - отвечает за установку обновлений на уже установленные приложения. Ключ `-y` означает что мы согласны на установку(удобно когда нужно массово обновить приложения)

Вот что мне выдало после выполнения данных команд:
```terminal
maki@lablearning:~$ sudo apt update && sudo apt upgrade -y
Hit:1 http://ru.archive.ubuntu.com/ubuntu noble InRelease
Hit:2 http://ru.archive.ubuntu.com/ubuntu noble-updates InRelease          
Hit:3 http://ru.archive.ubuntu.com/ubuntu noble-backports InRelease        
Hit:4 http://security.ubuntu.com/ubuntu noble-security InRelease           
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
All packages are up to date.
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
Calculating upgrade... Done
0 upgraded, 0 newly installed, 0 to remove and 0 not upgraded.
maki@lablearning:~$ 
```

Далее устанавливаем curl. Усттановка выполняется данной командой:
```terminal
sudo apt install curl -y 
```

Если у вас уже был установлен curl то вы увидите это:
```terminal
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
curl is already the newest version (8.5.0-2ubuntu10.7).
curl set to manually installed.
0 upgraded, 0 newly installed, 0 to remove and 0 not upgraded.
maki@lablearning:~$ 
```

### Основные команды APT
- `sudo apt update` — обновить список доступных пакетов.
- `sudo apt install <name>` — установить пакет.
- `sudo apt remove <name>` — удалить пакет.
[[Нужные команды для DevOps Engineer#^f35110| APT]]
### Создание скрипта

> [!example] Основы Bash
> Скрипт начинается с **Shebang**: `#!/bin/bash`. 
> Чтобы запустить файл как скрипт, нужны права на исполнение: `chmod +x script.sh`.

Далее создадим скрипт прямо в терминале, который проверяет время работы машины и подключенные диски `/dev` 
```terminal
cat << 'EOF' > ~/projects/learning/linux/check_system.sh
#!/bin/bash
echo "Checking system status for user: $USER"
uptime
df -h | grep '^/dev/'
EOF
```

После чего мы выдаем права на выполнение данного скрипта:
```terminal
chmod +x ~/projects/learning/linux/check_system.sh
```

Далее, переходим в директорию и выполняем данный скрипт:
```terminal
cd ~/projects/learning/linux/
./check_system.sh
```

И мы получаем такой вывод:
```terminal
Checking system status for user: maki
 17:01:46 up 2 days,  6:59,  2 users,  load average: 0.00, 0.01, 0.00
/dev/mapper/ubuntu--vg-ubuntu--lv   19G  4.6G   13G  27% /
/dev/sda2                          2.0G  104M  1.7G   6% /boot
maki@lablearning:~/projects/learning/linux$ 
```
## Задание

**Цель**: Научиться устанавливать софт и базово автоматизировать проверку его состояния. **Шаги**:

1. Установи веб-сервер **Nginx** через `apt`.
2. Напиши Bash-скрипт `service_manager.sh`, который:
    - Выводит фразу "Checking Nginx status..."
    - Выполняет команду `systemctl status nginx`.
    - Записывает текущую дату и статус в файл `~/service_logs.txt` (используй `>>` для дозаписи).
3. Сделай скрипт исполняемым и запусти его. **Ожидаемый результат**: Установленный Nginx и созданный файл лога с записью о его статусе. **Как проверить себя**: После запуска скрипта проверь содержимое файла: `cat ~/service_logs.txt`

### Выполнение
 
 Устанавливаем Nginx если он не установлен:
 ```terminal
 sudo apt install nginx
 ```

В моем случае он уже был установлен:
```terminal
maki@lablearning:~/projects/learning/linux$ sudo apt install nginx
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
nginx is already the newest version (1.24.0-2ubuntu7.6).
0 upgraded, 0 newly installed, 0 to remove and 0 not upgraded.
maki@lablearning:~/projects/learning/linux$ 
```

В ином случае рекомендую вызвать nginx и проверить версию, так мы будем знать что сервис видит наши команды и живет своей спокойной жизнью:
```terminal
nginx -v
```

Далее напишем скрипт по аналогии с практики:
```terminal
cat << 'SEX' > ~/projects/learning/linux/service_manager.sh
#!/bin/bash
echo "Checking status Nginx..."
systemctl status nginx | grep 'Loaded*'

systemctl status nginx | grep 'Active*'

STATUS=$(systemctl  is-active nginx)
echo "$(date '+%Y-%m-%d %H:%M:%S') - nginx: $STATUS" >> ~/service_logs.txt
SEX
```

Вот такой вывод у нас получился:
```terminal
Checking status Nginx...
     Loaded: loaded (/usr/lib/systemd/system/nginx.service; enabled; preset: enabled)
     Active: active (running) since Wed 2026-03-04 10:02:18 UTC; 2 days ago
```
Так же у нас записалось в наш созданный лог:
```terminal
maki@lablearning:~/projects/learning/linux$ cat ~/service_logs.txt 
2026-03-06 17:33:07 - nginx: active
```

## Контрольные вопросы

1. В чем разница между `apt update` и `apt upgrade`?
> 1. В чем разница между `apt update` и `apt upgrade`?
    > Update отвечает за обновления списка пакетов из доверенных репозиториев. 
2. Зачем в начале Bash-скрипта ставить `#!/bin/bash`?
3. Какой символ используется для перенаправления вывода команды в файл с полной перезаписью файла?
