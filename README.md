# Архитектура сетей.

## Теоретическая часть

Найти свободные подсети

```
Сегмент сети	IP-адресация	      Наименование	    число IP-адресов
------------------------------------------------------------------------
Сеть office1	192.168.2.0/26	    dev	                  64
Сеть office1	192.168.2.64/26	    test servers	      64
Сеть office1	192.168.2.128/26	managers	          64
Сеть office1	192.168.2.192/26    office hardware	      64
Сеть office2	192.168.1.0/25	    dev	                  128
Сеть office2	192.168.1.128/26	test servers	      64
Сеть office2	192.168.1.192/26	office hardware	      64
Сеть central	192.168.0.16/28	    нераспределенная	  16
Сеть central	192.168.0.128/25	нераспределенная	  128
Сеть central	192.168.0.48/28	    нераспределенная	  16
Сеть central	192.168.0.0/28	    directors	          16
Сеть central	192.168.0.32/28	    office hardware	      16
Сеть central	192.168.0.64/26	    wifi	              64
```
Указать broadcast адрес для каждой подсети

```
Сегмент сети	имя подсети	        IP-адресация	      broadcast-IP
-----------------------------------------------------------------------
Сеть office1	dev	                192.168.2.0/26	      192.168.2.63
Сеть office1	test servers	    192.168.2.64/26	      192.168.2.127
Сеть office1	managers	        192.168.2.128/26	  192.168.2.191
Сеть office1	office hardware	    192.168.2.192/26	  192.168.2.255
Сеть office2	dev	                192.168.1.0/25	      192.168.1.127
Сеть office2	test servers	    192.168.1.128/26	  192.168.1.191
Сеть office2	office hardware	    192.168.1.192/26	  192.168.1.255
Сеть central	нераспределенная	192.168.0.16/28	      192.168.0.31
Сеть central	нераспределенная	192.168.0.128/25	  192.168.0.255
Сеть central	нераспределенная	192.168.0.48/28	      192.168.0.63
Сеть central	directors	        192.168.0.0/28	      192.168.0.15
Сеть central	office hardware	    192.168.0.32/28	      192.168.0.47
Сеть central	wifi	            192.168.0.64/26	      192.168.0.127
```
## Развертывание стенда для демострации работы сетевой лаборатории

Стэнд состоит из хостовой машины под управлением ОС Ubuntu 20.04 на которой развернут Ansible и семи виртуальных машин CentOS/7 2004.01 (взял только одну ОС, т.к. проще отлаживать при условии необходимости качать виртуальные машины через VPN).

Машины с именами: `inetRouter`, `centralRouter`,  `office2Router`, `office1Router` выполняют роль роутеров.

Машины с именами: `centralServer`, `office1Server`,  `office2Server` выполняют роль серверов (клиентов) в подсетях.

В исходном `Vagrantfile` изменил объем выделяемой оперативной памяти, т.к. на хостовой машине всего 8 Gb. 

Разворачиваем инфраструктуру в `Vagrant` исключительно через `Ansible`.

Все коментарии по каждому блоку указаны в тексте `Playbook` - `lan.yml`.

## Результат работы

Вывод маршрутов и пингов IP адресов для выхода в интернет и доступа в смежные офисы на `office1Server`:
```
[root@office1Server vagrant]# ip route get 8.8.8.8
8.8.8.8 via 192.168.2.129 dev eth1 src 192.168.2.130 
    cache 
[root@office1Server vagrant]# ip route get 192.168.0.2
192.168.0.2 via 192.168.2.129 dev eth1 src 192.168.2.130 
    cache 
[root@office1Server vagrant]# ip route get 192.168.1.2
192.168.1.2 via 192.168.2.129 dev eth1 src 192.168.2.130 
    cache 
[root@office1Server vagrant]# ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=57 time=162 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=57 time=203 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=57 time=191 ms
64 bytes from 8.8.8.8: icmp_seq=4 ttl=57 time=172 ms
^C
--- 8.8.8.8 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3096ms
rtt min/avg/max/mdev = 162.758/182.628/203.527/15.926 ms
[root@office1Server vagrant]# ping 192.168.0.2
PING 192.168.0.2 (192.168.0.2) 56(84) bytes of data.
64 bytes from 192.168.0.2: icmp_seq=1 ttl=62 time=0.939 ms
64 bytes from 192.168.0.2: icmp_seq=2 ttl=62 time=3.09 ms
64 bytes from 192.168.0.2: icmp_seq=3 ttl=62 time=3.07 ms
64 bytes from 192.168.0.2: icmp_seq=4 ttl=62 time=2.95 ms
^C
--- 192.168.0.2 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3077ms
rtt min/avg/max/mdev = 0.939/2.517/3.099/0.912 ms
[root@office1Server vagrant]# ping 192.168.1.2
PING 192.168.1.2 (192.168.1.2) 56(84) bytes of data.
64 bytes from 192.168.1.2: icmp_seq=1 ttl=61 time=4.33 ms
64 bytes from 192.168.1.2: icmp_seq=2 ttl=61 time=3.60 ms
64 bytes from 192.168.1.2: icmp_seq=3 ttl=61 time=3.62 ms
64 bytes from 192.168.1.2: icmp_seq=4 ttl=61 time=4.51 ms
^C
--- 192.168.1.2 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3046ms
rtt min/avg/max/mdev = 3.604/4.018/4.512/0.410 ms
```
Вывод маршрутов и пингов IP адресов для выхода в интернет и доступа в смежные офисы на `centralServer`:

```
[root@centralServer vagrant]# ip route get 8.8.8.8        
8.8.8.8 via 192.168.0.1 dev eth1 src 192.168.0.2 
    cache 
[root@centralServer vagrant]# ip route get 192.168.2.130
192.168.2.130 via 192.168.0.1 dev eth1 src 192.168.0.2 
    cache 
[root@centralServer vagrant]# ip route get 192.168.1.2  
192.168.1.2 via 192.168.0.1 dev eth1 src 192.168.0.2 
    cache 
[root@centralServer vagrant]# ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=59 time=166 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=59 time=158 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=59 time=153 ms
64 bytes from 8.8.8.8: icmp_seq=4 ttl=59 time=137 ms
64 bytes from 8.8.8.8: icmp_seq=5 ttl=59 time=139 ms
^C
--- 8.8.8.8 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4020ms
rtt min/avg/max/mdev = 137.458/151.314/166.711/11.193 ms
[root@centralServer vagrant]# ping 192.168.2.130
PING 192.168.2.130 (192.168.2.130) 56(84) bytes of data.
64 bytes from 192.168.2.130: icmp_seq=1 ttl=62 time=2.94 ms
64 bytes from 192.168.2.130: icmp_seq=2 ttl=62 time=3.00 ms
64 bytes from 192.168.2.130: icmp_seq=3 ttl=62 time=2.83 ms
64 bytes from 192.168.2.130: icmp_seq=4 ttl=62 time=3.00 ms
^C
--- 192.168.2.130 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3011ms
rtt min/avg/max/mdev = 2.836/2.948/3.009/0.080 ms
[root@centralServer vagrant]# ping 192.168.1.2
PING 192.168.1.2 (192.168.1.2) 56(84) bytes of data.
64 bytes from 192.168.1.2: icmp_seq=1 ttl=62 time=2.84 ms
64 bytes from 192.168.1.2: icmp_seq=2 ttl=62 time=2.99 ms
64 bytes from 192.168.1.2: icmp_seq=3 ttl=62 time=2.85 ms
64 bytes from 192.168.1.2: icmp_seq=4 ttl=62 time=2.71 ms
^C
--- 192.168.1.2 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3044ms
rtt min/avg/max/mdev = 2.719/2.854/2.997/0.105 ms
[root@centralServer vagrant]# ping otus.ru    
PING otus.ru (188.114.98.224) 56(84) bytes of data.
64 bytes from 188.114.98.224 (188.114.98.224): icmp_seq=1 ttl=59 time=221 ms
64 bytes from 188.114.98.224 (188.114.98.224): icmp_seq=2 ttl=59 time=210 ms
64 bytes from 188.114.98.224 (188.114.98.224): icmp_seq=3 ttl=59 time=189 ms
64 bytes from 188.114.98.224 (188.114.98.224): icmp_seq=4 ttl=59 time=126 ms
^C
--- otus.ru ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3055ms
rtt min/avg/max/mdev = 126.854/187.220/221.571/36.673 ms

```
Вывод маршрутов и пингов IP адресов для выхода в интернет и доступа в смежные офисы на `office2Server`:
```
[root@office2Server vagrant]# ip route get 8.8.8.8
8.8.8.8 via 192.168.1.1 dev eth1 src 192.168.1.2 
    cache 
[root@office2Server vagrant]# ip route get 192.168.0.2
192.168.0.2 via 192.168.1.1 dev eth1 src 192.168.1.2 
    cache 
[root@office2Server vagrant]# ip route get 192.168.2.130
192.168.2.130 via 192.168.1.1 dev eth1 src 192.168.1.2 
    cache 
[root@office2Server vagrant]# ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=57 time=153 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=57 time=171 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=57 time=124 ms
^C
--- 8.8.8.8 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2010ms
rtt min/avg/max/mdev = 124.354/149.824/171.775/19.518 ms
[root@office2Server vagrant]# ping 192.168.0.2
PING 192.168.0.2 (192.168.0.2) 56(84) bytes of data.
64 bytes from 192.168.0.2: icmp_seq=1 ttl=62 time=2.93 ms
64 bytes from 192.168.0.2: icmp_seq=2 ttl=62 time=2.99 ms
64 bytes from 192.168.0.2: icmp_seq=3 ttl=62 time=3.19 ms
64 bytes from 192.168.0.2: icmp_seq=4 ttl=62 time=2.91 ms
^C
--- 192.168.0.2 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3018ms
rtt min/avg/max/mdev = 2.915/3.010/3.198/0.124 ms
[root@office2Server vagrant]# ping 192.168.2.130
PING 192.168.2.130 (192.168.2.130) 56(84) bytes of data.
64 bytes from 192.168.2.130: icmp_seq=1 ttl=61 time=3.71 ms
64 bytes from 192.168.2.130: icmp_seq=2 ttl=61 time=4.10 ms
64 bytes from 192.168.2.130: icmp_seq=3 ttl=61 time=3.45 ms
64 bytes from 192.168.2.130: icmp_seq=4 ttl=61 time=3.69 ms
^C
--- 192.168.2.130 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3007ms
rtt min/avg/max/mdev = 3.456/3.740/4.104/0.236 ms
[root@office2Server vagrant]# ping otus.ru      
PING otus.ru (188.114.98.224) 56(84) bytes of data.
64 bytes from 188.114.98.224 (188.114.98.224): icmp_seq=1 ttl=57 time=204 ms
64 bytes from 188.114.98.224 (188.114.98.224): icmp_seq=2 ttl=57 time=226 ms
64 bytes from 188.114.98.224 (188.114.98.224): icmp_seq=3 ttl=57 time=143 ms
64 bytes from 188.114.98.224 (188.114.98.224): icmp_seq=4 ttl=57 time=162 ms
^C
--- otus.ru ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3011ms
rtt min/avg/max/mdev = 143.531/184.087/226.181/32.873 ms

```
## P.S.

`handlers restart network` в `Playbook` при разворачиваем инфраструктуру в Vagrant на некоторых виртуальных машинах не всегдаа отрабатывал корректно. Ошибок в процессе развертывания не выводилось. После выполнения команды `systemctl restart network` в терминале самой виртуальной машины все начинало работать штатно.