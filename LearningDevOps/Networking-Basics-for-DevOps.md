> [!note] Определение
> **Компьютерная сеть** — это система связи, позволяющая серверам и приложениям обмениваться данными. Для DevOps критично понимание модели OSI и протокола TCP/IP.

##  Краткое объяснение
В DevOps сеть — это клей, который соединяет сервисы. Если в администрировании Windows мы часто полагаемся на графические оснастки DHCP/DNS, то здесь нам нужно понимать, как пакет проходит от клиента до контейнера. Мы разберем, как сервер понимает, кому адресован запрос, и как работают порты.

## Полезные команды диагностики
- `ip addr` — просмотр IP и статуса интерфейсов.
- `ss -tunlp` — список слушающих портов (sockets).
- `dig` / `nslookup` — диагностика DNS.
- `curl -I` — проверка HTTP-заголовков.

##  Ключевые понятия темы

- **IP-адрес и Маска подсети**: как отделяется адрес сети от адреса хоста.
- **Default Gateway**: шлюз, через который сервер «ходит» в интернет.
- **DNS (Domain Name System)**: преобразование имен (google.com) в IP.
- **Порты (Ports)**: «двери» сервера (80 — HTTP, 443 — HTTPS, 22 — SSH).
- **Модель OSI (L3, L4, L7)**: уровни, на которых работают роутеры (L3), балансировщики (L4/L7) и приложения (L7)    

> [!example] Уровни OSI (упрощенно)
> - **L3 (Сетевой)**: IP-адреса, маршрутизация.
> - **L4 (Транспортный)**: TCP/UDP, порты.
> - **L7 (Прикладной)**: HTTP, DNS, TLS.

## Практика (минимальный рабочий пример)

Исследуем сетевое окружение твоего сервера:

```
ip addr                     # Посмотреть IP-адрес и сетевые интерфейсы
ip route                    # Узнать адрес шлюза по умолчанию (default via ...)
ss -tunlp                   # Посмотреть, какие порты "слушают" программы (найди там nginx на 80 порту)
cat /etc/resolv.conf        # Посмотреть, какой DNS-сервер использует твоя система
curl -I http://localhost    # Сделать HTTP-запрос к самому себе и увидеть заголовки Nginx
```

Вывод команды `ip addr`:
```terminal
maki@lablearning:~$ ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host noprefixroute 
       valid_lft forever preferred_lft forever
2: ens160: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:0c:29:ff:6d:2c brd ff:ff:ff:ff:ff:ff
    altname enp3s0
    inet 192.168.101.131/24 metric 100 brd 192.168.101.255 scope global dynamic ens160
       valid_lft 32424sec preferred_lft 32424sec
    inet6 fdb5:757:53d2:0:20c:29ff:feff:6d2c/64 scope global dynamic mngtmpaddr noprefixroute 
       valid_lft 5354sec preferred_lft 2654sec
    inet6 fd3f:b49d:94d5::f9f/128 scope global dynamic noprefixroute 
       valid_lft 26993sec preferred_lft 26993sec
    inet6 fd3f:b49d:94d5:0:20c:29ff:feff:6d2c/64 scope global mngtmpaddr noprefixroute 
       valid_lft forever preferred_lft forever
    inet6 fe80::20c:29ff:feff:6d2c/64 scope link 
       valid_lft forever preferred_lft forever
maki@lablearning:~$ 
```

Вывод команды `ip route`:
```terminal
maki@lablearning:~$ ip route
default via 192.168.101.1 dev ens160 proto dhcp src 192.168.101.131 metric 100 
192.168.101.0/24 dev ens160 proto kernel scope link src 192.168.101.131 metric 100 
192.168.101.1 dev ens160 proto dhcp scope link src 192.168.101.131 metric 100 
maki@lablearning:~$ 
```

Вывод команды `ss -tunlp`:
```terminal
maki@lablearning:~$ ss -tunlp
Netid        State         Recv-Q        Send-Q                                   Local Address:Port                  Peer Address:Port        Process        
udp          UNCONN        0             0                                           127.0.0.54:53                         0.0.0.0:*                          
udp          UNCONN        0             0                                        127.0.0.53%lo:53                         0.0.0.0:*                          
udp          UNCONN        0             0                               192.168.101.131%ens160:68                         0.0.0.0:*                          
udp          UNCONN        0             0                    [fe80::20c:29ff:feff:6d2c]%ens160:546                           [::]:*                          
tcp          LISTEN        0             4096                                     127.0.0.53%lo:53                         0.0.0.0:*                          
tcp          LISTEN        0             4096                                        127.0.0.54:53                         0.0.0.0:*                          
tcp          LISTEN        0             4096                                           0.0.0.0:23784                      0.0.0.0:*                          
tcp          LISTEN        0             511                                            0.0.0.0:80                         0.0.0.0:*                          
tcp          LISTEN        0             4096                                              [::]:23784                         [::]:*                          
tcp          LISTEN        0             511                                               [::]:80                            [::]:*                          
maki@lablearning:~$ 
```

Вывод команды `cat /etc/resolv.conf`:
```terminal
maki@lablearning:~$ cat /etc/resolv.conf
# This is /run/systemd/resolve/stub-resolv.conf managed by man:systemd-resolved(8).
# Do not edit.
#
# This file might be symlinked as /etc/resolv.conf. If you're looking at
# /etc/resolv.conf and seeing this text, you have followed the symlink.
#
# This is a dynamic resolv.conf file for connecting local clients to the
# internal DNS stub resolver of systemd-resolved. This file lists all
# configured search domains.
#
# Run "resolvectl status" to see details about the uplink DNS servers
# currently in use.
#
# Third party programs should typically not access this file directly, but only
# through the symlink at /etc/resolv.conf. To manage man:resolv.conf(5) in a
# different way, replace this symlink by a static file or a different symlink.
#
# See man:systemd-resolved.service(8) for details about the supported modes of
# operation for /etc/resolv.conf.

nameserver 127.0.0.53
options edns0 trust-ad
search lan
maki@lablearning:~$ 
```

Вывод команды `curl -I http://loclhost:80`
```terminal
maki@lablearning:~$ curl -I http://localhost:80
HTTP/1.1 200 OK
Server: nginx/1.24.0 (Ubuntu)
Date: Tue, 10 Mar 2026 17:30:57 GMT
Content-Type: text/html
Content-Length: 635
Last-Modified: Sat, 28 Feb 2026 12:42:04 GMT
Connection: keep-alive
ETag: "69a2e29c-27b"
Accept-Ranges: bytes

maki@lablearning:~$ 
```

## Задание

**Цель**: Разобраться в сетевой конфигурации своего окружения. **Шаги**:

1. Узнай свой **локальный IP** внутри ВМ.
	 Можно узнать при помощи `ip addr`.
	 В моем случае это интерфейс ens160:
	 ens160: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:0c:29:ff:6d:2c brd ff:ff:ff:ff:ff:ff
    altname enp3s0
    inet 192.168.101.131/24 metric 100 brd 192.168.101.255 scope global dynamic ens160
2. Узнай свой **внешний IP** (используй `curl ifconfig.me`).
	 Получил такой ответ: 
	 maki@lablearning:~$ curl ifconfig.me 80.87.144.133
3. Проверь, через какой DNS-сервер разрешается имя `google.com` (команда `dig google.com` или `nslookup google.com`).
	 Зачастую я использую `nslookup`.
	 maki@lablearning:~$ dig google.com
	 ; <<>> DiG 9.18.39-0ubuntu0.24.04.2-Ubuntu <<>> google.com
	 ;; global options: +cmd
	 ;; Got answer:
	 ;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 27367
	 ;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1
	 ;; OPT PSEUDOSECTION:
	 ; EDNS: version: 0, flags:; udp: 65494
	 ;; QUESTION SECTION:
	 ;google.com.                    IN      A
	 ;; ANSWER SECTION:
	 google.com.             261     IN      A       142.251.143.142
	 ;; Query time: 437 msec
	 ;; SERVER: 127.0.0.53#53(127.0.0.53) (UDP)
	 ;; WHEN: Tue Mar 10 17:35:19 UTC 2026
	 ;; MSG SIZE  rcvd: 55
	 maki@lablearning:~$ nslookup google.com
	 Server:         127.0.0.53
	 Address:        127.0.0.53#53
	 Non-authoritative answer:
	 Name:   google.com
	 Address: 142.251.143.142
	 Name:   google.com
	 Address: ::
	 maki@lablearning:~$ 
4. Попробуй просканировать порты своей ВМ со своего основного Windows-компьютера (используй команду `Test-NetConnection -ComputerName IP_ТВОЕЙ_ВМ -Port 80` в PowerShell). **Ожидаемый результат**: Список IP, понимание, какой DNS отвечает за запросы, и подтверждение доступности Nginx снаружи. **Как проверить себя**: Если `Test-NetConnection` на Windows пишет `TcpTestSucceeded : True`, значит, твой Firewall (UFW) настроен верно и сеть работает.
	 PS C:\WINDOWS\System32> Test-NetConnection -ComputerName 192.168.101.131 -Port 80
	 ComputerName     : 192.168.101.131
	 RemoteAddress    : 192.168.101.131
	 RemotePort       : 80
	 InterfaceAlias   : Wi-Fi
	 SourceAddress    : 192.168.101.201
	 TcpTestSucceeded : True
	 PS C:\WINDOWS\System32>

## Контрольные вопросы

1. На каком уровне модели OSI работает протокол **HTTP** (L3, L4 или L7)?
	 На L7
2. В чем разница между портами **80** и **443**?
	 Порт 80 олтвечает за HTTP, порт 443 за HTTP/S
3. Какая запись в файле `/etc/hosts` позволит тебе обращаться к своей ВМ по имени `myserver.local`?
	 192.168.101.131    myserver.local
	 На windows наоборот пишет IP и DNS
### Чек-лист
- [ ] Знаю свой локальный и внешний IP.
- [ ] Понимаю, как работает разрешение имен (DNS).
- [ ] Умею проверять доступность портов.

**Связанные заметки**: [[Linux Monitoring]], [[Nginx as Proxy]], [[Linux Monitoring & Diagnostics]], [[Пакетный менеджер и Веб-сервер (Nginx)]]


**Tags**: #networks #linux #dns #osi