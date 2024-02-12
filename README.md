# 1 ALL
Если в задании не будут использоваться встроенные репозитории, а будет возможность скачивать все пакеты из интернета, необходимо отключить проверку пакетов через cdrom зайдя по пути Nano /etc/apt/sources.list и Закоментировать находящуюся там строку.  
### Name   
```
hostnamectl set-hostname Имя устройства  
Newgrp
```
### IP
|Name|Ipv4|mask|Def_get|Ipv6/mask|Def_getv6|
|---|---|---|---|---|---|
|CLI-ISP|192.168.0.2|255.255.255.0|192.168.0.1|2001::3:2/120|2001::3:1/120|!
|CLI-HQR|-|-|-|-|-|
|ISP-ClI|192.168.0.1|255.255.255.0|-|2001::3:1/120|-|
|ISP-HQR|10.10.10.2|255.255.255.252|-|2001::7:2/126|-| 
|ISP-BRR|10.10.10.6|255.255.255.252|-|2001::7:6/126|-|
|HQR-HQSRV|192.168.1.1|255.255.255.192|-|2001::1:1/122|-| 
|HQR-ISP|10.10.10.1|255.255.255.252|-|2001::7:1/126|-|
|HQR-CLI|-|-|-|-|-|
|HQSRV-HQR|192.168.1.2|255.255.255.192|192.168.1.1|2001::1:2/122|2001::1:1/122|
|BRR-BRSRV|192.168.2.1|255.255.255.240|-|2001::2:1/124|-|
|BRR- ISP|10.10.10.5|255.255.255.252|-|2001::7:5/126|-|
|BRSRV-BRR|192.168.2.2|255.255.255.240|10.10.10.5|2001::2:2/124|2001::2:1/124|
---
![maskv4](https://myeditor.ru/wp-content/uploads/b/8/3/b83c1b85ee91682121df78aca1e4576f.png)  
```
Ip a  
nano /etc/network/interfaces  
```
Пример v4:  
```  
auto ens256  
iface ens256 inet static   
address 10.10.10.1  
netmask 255. 255.255.0    
gateway 10.10.10.2  
```
Пример v6: 
```
auto ens256 
iface ens256 ine6 static 
address 2001::7:1 
netmask 126 
gateway 2001::7:2
```
после перезагружаем сеть```systemctl restart networking```
# 2 BRR HQR ISP
```
apt install frr 
nano /etc/frr/daemons
```
изменить параметры на YES для протокола ospfd и ospf6d  
### Настройка OSPF v4
```
Vtysh 
Conf t  
router ospf 
```
|Name|id|network|area|
|---|---|---|---|
|BRR-ISP|2.2.2.2|10.10.10.4/30|0|
|BRR-BRSRV|2.2.2.2|192.168.2.0/28|2| 
|HQR-ISP|3.3.3.3|10.10.10.0/30|0|
|HQR-HQRSRV|3.3.3.3|192.168.1.0/26|3|
|ISP-BRR|4.4.4.4|10.10.10.4/30|0| 
|ISP-HQR|4.4.4.4|10.10.10.0/30|0| 
|ISP-CLI|4.4.4.4|192.168.3.0/24|4|
---
Пример:
```
router ospf  
ospf router-id 2.2.2.2  
network 10.10.10.4/30 area 0  
network 192.168.2.0/28 area 2
```
После завершения конфигурации в frr написать write
```nano /etc/sysctl.conf```  
переменную ```net.ipv4.ip_forward=1``` и ```net.ipv6.conf.all.forwarding=1``` необходимо раскоментить и сохранинть измнения в файле, и применить изменения командой
```sysctl –p ```  
При каждой перезагрузки прописывать sysctl –p 
### Настройка OSPF v6
```
Vtysh
Conf t  
router ospf6  
```
|Name|id|area|range/mask|
|---|---|---|---|
|BRR-ISP|0.0.0.2|0.0.0.0|2001::7:4/126|
|BRR-BRSRV|0.0.0.2|0.0.0.5|2001::2:0/124| 
|HQR-ISP|0.0.0.3|0.0.0.0|2001::7:0/126|
|HQR-HQRSRV|0.0.0.3|0.0.0.6|2001::1:0/122|
|ISP-BRR|0.0.0.4|0.0.0.0|2001::7:4/126| 
|ISP-HQR|0.0.0.4|0.0.0.0|2001::7:0/126| 
|ISP-CLI|0.0.0.4|0.0.0.7|2001::3:0/120|
---  
Пример:  
```
router ospf6  
ospf6 router-id 0.0.0.1  
area 0.0.0.0 range 2001::1:0/122  
area 0.0.0.0 range 2001::7:0/126  
```
