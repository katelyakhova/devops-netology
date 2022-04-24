# Домашнее задание к занятию "3.7. Компьютерные сети, лекция 2"

1. Проверьте список доступных сетевых интерфейсов на вашем компьютере. Какие команды есть для этого в Linux и в Windows?

```bash
$ ip -br link show
lo               UNKNOWN        00:00:00:00:00:00 <LOOPBACK,UP,LOWER_UP> 
enp58s0          DOWN           f4:ee:08:e1:46:b1 <NO-CARRIER,BROADCAST,MULTICAST,UP> 
wlp0s20f3        UP             4c:77:cb:5d:0b:dd <BROADCAST,MULTICAST,UP,LOWER_UP> 
```

2. Какой протокол используется для распознавания соседа по сетевому интерфейсу? Какой пакет и команды есть в Linux для этого?

Протокол LLDP используется для обмена информацией между соседними устройствами
В Ubuntu используется пакет lldpd.
Посмотреть своих lldp соседей можно
```bash
$ lldpctl
-------------------------------------------------------------------------------
LLDP neighbors:
-------------------------------------------------------------------------------

```

3. Какая технология используется для разделения L2 коммутатора на несколько виртуальных сетей? Какой пакет и команды есть в Linux для этого? Приведите пример конфига.

Для разделения коммутатора на несколько виртуальных сетей используется технология VLAN
В  Ubuntu есть пакет vlan
Настройки подынтерфейсов VLANов в Ubuntu точно так же, как и для сетевых интерфейсов, 
указываются в файле `/etc/network/interfaces`
Пример:
```bash
auto vlan1400
iface vlan1400 inet static
        address 192.168.1.1
        netmask 255.255.255.0
        vlan_raw_device eth0
```
1400 указывает какой VLAN ID должен использоваться.

4. Какие типы агрегации интерфейсов есть в Linux? Какие опции есть для балансировки нагрузки? Приведите пример конфига.

Типы агрегации интерфейсов в Linux:  
mode=0 (balance-rr) - последовательно кидает пакеты, с первого по последний интерфейс.  
mode=1 (active-backup) - один из интерфейсов активен. Если активный интерфейс выходит из строя, 
другой интерфейс заменяет активный. Не требует дополнительной настройки коммутатора  
mode=2 (balance-xor) - передачи распределяются между интерфейсами на основе формулы 
((MAC-адрес источника) XOR (MAC-адрес получателя)) % число интерфейсов. 
Один и тот же интерфейс работает с определённым получателем. 
Режим даёт балансировку нагрузки и отказоустойчивость.  
mode=3 (broadcast) - все пакеты на все интерфейсы  
mode=4 (802.3ad) Link Agregation — IEEE 802.3ad.  
mode=5 (balance-tlb) - входящие пакеты принимаются только активным сетевым интерфейсом, 
исходящий распределяется в зависимости от текущей загрузки каждого интерфейса.  
mode=6 (balance-alb) - тоже самое что 5, только входящий трафик тоже распределяется между интерфейсами. 

5. Сколько IP адресов в сети с маской /29 ?   
8 IP адресов, но 0 - это подсеть, а 7 - это broadcast. Итого остается 6 используемых IP адресов.

```bash
$ ipcalc 10.10.10.0/29
Address:   10.10.10.0           00001010.00001010.00001010.00000 000
Netmask:   255.255.255.248 = 29 11111111.11111111.11111111.11111 000
Wildcard:  0.0.0.7              00000000.00000000.00000000.00000 111
=>
Network:   10.10.10.0/29        00001010.00001010.00001010.00000 000
HostMin:   10.10.10.1           00001010.00001010.00001010.00000 001
HostMax:   10.10.10.6           00001010.00001010.00001010.00000 110
Broadcast: 10.10.10.7           00001010.00001010.00001010.00000 111
Hosts/Net: 6                     Class A, Private Internet
```

Сколько /29 подсетей можно получить из сети с маской /24.  
В сети с маской /24 256 хостов. В сети с маской /29 8 хостов. 
Значит в сети /24 можно организовать 256/8=32 подсети /29

```bash
$ ipcalc 10.10.10.0/24
Address:   10.10.10.0           00001010.00001010.00001010. 00000000
Netmask:   255.255.255.0 = 24   11111111.11111111.11111111. 00000000
Wildcard:  0.0.0.255            00000000.00000000.00000000. 11111111
=>
Network:   10.10.10.0/24        00001010.00001010.00001010. 00000000
HostMin:   10.10.10.1           00001010.00001010.00001010. 00000001
HostMax:   10.10.10.254         00001010.00001010.00001010. 11111110
Broadcast: 10.10.10.255         00001010.00001010.00001010. 11111111
Hosts/Net: 254                   Class A, Private Internet
```

Приведите несколько примеров /29 подсетей внутри сети 10.10.10.0/24.
10.10.10.0/29
10.10.10.8/29
10.10.10.16/29

6. Задача: вас попросили организовать стык между 2-мя организациями. Диапазоны 10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16 уже заняты. Из какой подсети допустимо взять частные IP адреса? Маску выберите из расчета максимум 40-50 хостов внутри подсети.

100.64.0.0 — 100.127.255.255 (маска подсети 255.192.0.0 или /10) - 
Данная подсеть рекомендована согласно RFC 6598 для использования в качестве адресов 
для CGN (Carrier-Grade NAT).

```bash
$ ipcalc 100.64.0.0/10 -s 50
Address:   100.64.0.0           01100100.01 000000.00000000.00000000
Netmask:   255.192.0.0 = 10     11111111.11 000000.00000000.00000000
Wildcard:  0.63.255.255         00000000.00 111111.11111111.11111111
=>
Network:   100.64.0.0/10        01100100.01 000000.00000000.00000000
HostMin:   100.64.0.1           01100100.01 000000.00000000.00000001
HostMax:   100.127.255.254      01100100.01 111111.11111111.11111110
Broadcast: 100.127.255.255      01100100.01 111111.11111111.11111111
Hosts/Net: 4194302               Class A

1. Requested size: 50 hosts
Netmask:   255.255.255.192 = 26 11111111.11111111.11111111.11 000000
Network:   100.64.0.0/26        01100100.01000000.00000000.00 000000
HostMin:   100.64.0.1           01100100.01000000.00000000.00 000001
HostMax:   100.64.0.62          01100100.01000000.00000000.00 111110
Broadcast: 100.64.0.63          01100100.01000000.00000000.00 111111
Hosts/Net: 62                    Class A

Needed size:  64 addresses.
Used network: 100.64.0.0/26
Unused:
100.64.0.64/26
100.64.0.128/25
100.64.1.0/24
100.64.2.0/23
100.64.4.0/22
100.64.8.0/21
100.64.16.0/20
100.64.32.0/19
100.64.64.0/18
100.64.128.0/17
100.65.0.0/16
100.66.0.0/15
100.68.0.0/14
100.72.0.0/13
100.80.0.0/12
100.96.0.0/11
```
Итого: можно выбрать 100.64.0.0/26

8. Как проверить ARP таблицу в Linux, Windows? Как очистить ARP кеш полностью? Как из ARP таблицы удалить только один нужный IP?
В Linux есть утилита `arp`. В ней следующие опции:

```bash
        -a                       display (all) hosts in alternative (BSD) style
        -e                       display (all) hosts in default (Linux) style
        -d, --delete             delete a specified entry
```

Очистить ARP кеш полностью можно командой `ip neigh flush all`