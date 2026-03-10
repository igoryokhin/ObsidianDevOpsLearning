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
```
### Чек-лист
- [ ] Знаю свой локальный и внешний IP.
- [ ] Понимаю, как работает разрешение имен (DNS).
- [ ] Умею проверять доступность портов.

**Связанные заметки**: [[Linux Monitoring]], [[Nginx as Proxy]], [[Linux Monitoring & Diagnostics]], [[Пакетный менеджер и Веб-сервер (Nginx)]]


**Tags**: #networks #linux #dns #osi