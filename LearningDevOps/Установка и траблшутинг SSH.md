
При установке чистого образа Ubuntu не устанавливается сервер SSH. Для этого нам потребуется проверить есть ли вообще SSH сервер на нашей виртуалке
# Траблшутитнг

выполняем команду systemctl для того чтобы понять какой статус у службы openssh:

```terminal
sudo systemctl status ssh
```
[[Нужные команды для DevOps Engineer#Systemctl|systemctl]]
Если данная служба есть в системе и она не активна то выполняем команду для активации ssh сервера:
```terminal
sudo systemctl enable --now ssh
```
[[Нужные команды для DevOps Engineer#Systemctl|systemctl]]
Теперь мы проверяем порт 22 на сетевом экране и активируем его:
```terminal
sudo ufw allow ssh
# Или можно указать порт напрямую:
sudo ufw allow 22/tcp
```
[[Нужные команды для DevOps Engineer#UFW(Сетевой экран/FireWall)|UFW]]
# Установка SSH сервера

Для установки нам понадобаиться установить SSH сервер, обычно используют openssh:
```terminal
sudo apt update
sudo apt install openssh-server -y
```
[[Нужные команды для DevOps Engineer#Advanced Package Tool|apt]]
после установки проверяем активность службы:
```terminal
sudo systemctl status ssh
```
[[Нужные команды для DevOps Engineer#Systemctl|systemctl]]
Если он активен то все у нас получилось! В ином случае придется его активировать данной командой:
```terminal
sudo systemctl enable --now ssh
```
[[Нужные команды для DevOps Engineer#Systemctl|systemctl]]
После восстановления/установки SSH сервера, нам нужно защитить от хакеров авторизацию на сервер. Мы будем создавать SSH ключ для авторизации: [[Создание SSH ключа на Windows]]


#### Проверка валидности конфига
Для проверки синтаксиса и корректности конфига нужно выполнить:
```terminal
sudo sshd -t
```

Если тишина — конфиг валидный.  
Если ошибка — он покажет строку.

#ssh #fix #фикс 