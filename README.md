# Домашнее задание к занятию 2 «Кластеризация и балансировка нагрузки»

**Студент:** Евгений Репин

## Задание 1. Балансировка Round-robin на 4 уровне

### Конфигурационный файл HAProxy

```haproxy
global
    log /dev/log local0
    user haproxy
    group haproxy
    daemon

defaults
    log global
    mode http
    timeout connect 5000
    timeout client 50000
    timeout server 50000

listen stats
    bind :8080
    stats enable
    stats uri /stats
    stats refresh 5s

frontend my_frontend
    bind *:80
    mode tcp
    default_backend my_servers

backend my_servers
    mode tcp
    balance roundrobin
    server server1 127.0.0.1:8001 check
    server server2 127.0.0.1:8002 check
```
Результат перенаправления запросов

Запросы чередуются между серверами:

https://skrin/Zadanie1.png

##Задание 2. Weighted Round Robin на 7 уровне
###Конфигурационный файл HAProxy

```
    global
    log /dev/log local0
    user haproxy
    group haproxy
    daemon

defaults
    log global
    mode http
    timeout connect 5000
    timeout client 50000
    timeout server 50000

listen stats
    bind :8080
    stats enable
    stats uri /stats
    stats refresh 5s

frontend http_frontend
    bind *:80
    mode http
    acl is_example_local hdr(host) -i example.local
    use_backend weighted_servers if is_example_local
    default_backend default_servers

backend weighted_servers
    mode http
    balance roundrobin
    option httpchk GET /
    server s1 127.0.0.1:9011 weight 2 check
    server s2 127.0.0.1:9012 weight 3 check
    server s3 127.0.0.1:9013 weight 4 check

backend default_servers
    mode http
    server default 127.0.0.1:9081 check
```

Результат с доменом example.local

9 запросов распределяются с весами 2:3:4:

https://skrin/Zadanie2.png
Результат без домена example.local

Запрос без домена уходит на default_server:

https://skrin/Zadanie2.2.png
Статистика с весами

https://skrin/Zadanie2.3.png
