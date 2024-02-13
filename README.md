# 1 ALL (Name/IPv4/IPv6)  
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
# 2 BRR HQR ISP (FRR-OSPF/L3)  
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
```exit```
После завершения конфигурации в frr написать ```write```
```nano /etc/sysctl.conf```  
переменную ```net.ipv4.ip_forward=1``` и ```net.ipv6.conf.all.forwarding=1``` необходимо раскоментить и сохранинть измнения в файле, и применить изменения командой
```sysctl –p ```  
```diff
-При каждой перезагрузки прописывать sysctl –p 
```  
### Настройка OSPF v6
```
Vtysh
Conf t  
router ospf6  
```
|Name|id|area|range/mask|
|---|---|---|---|
|BRR-ISP|0.0.0.2|0.0.0.0|2001::7:4/126|
|BRR-BRSRV|0.0.0.2|0.0.0.0|2001::2:0/124| 
|HQR-ISP|0.0.0.3|0.0.0.0|2001::7:0/126|
|HQR-HQRSRV|0.0.0.3|0.0.0.0|2001::1:0/122|
|ISP-BRR|0.0.0.4|0.0.0.0|2001::7:4/126| 
|ISP-HQR|0.0.0.4|0.0.0.0|2001::7:0/126| 
|ISP-CLI|0.0.0.4|0.0.0.0|2001::3:0/120|
---  
Пример:  
```
router ospf6  
ospf6 router-id 0.0.0.1  
area 0.0.0.0 range 2001::1:0/122  
area 0.0.0.0 range 2001::7:0/126  
```
```exit```
```
interface ens224
ipv6 ospf6 area 0.0.0.0
exit
``` 
После завершения конфигурации в frr написать ```write```
L3 Пример: 1 соединения ipv4/mask and ipv6/mask  
# 3 HQ-R (DHCP)  
```  
apt install isc-dhcp-server  
nano /etc/default/isc-dhcp-server
```  
если в сети подразумевается DHCP-relay ,то  
1 интерфейс в сторону клиента, 2 интерфейс в сторону запроса  
```
INTERFACESv4="ens224 ens256"   
INTERFACESv6="ens224 ens256"
```  
---
### IPv4
``` nano /etc/dhcp/dhcpd.conf ```  
Пример DHCP для ipv4 без Relay:  
```
default-lease-time 600;  
max-lease-time 7200;  
ddns-updates on;  
ddns-update-style interim;  
authoritative;
  
subnet 192.168. 1.0 netmask 255.255.255. 192 {
 range 192. 168.1.3 192.168.1.62;
 option routers 192.168.1.1;
 option domain-name "hq.work";
 option domain-name-servers 192.168.1.2;
}
```  
Где:  
```ddns-update-style interim``` — способ автообновления базы dns  
```authoritative``` — делает сервер доверенным  
```subnet``` — указание сети  
```range — пул адресов```  
```option routers — шлюз по умолчанию```  
```diff
-Примечание: после каждого изменения конфигурации необходимо
-перезагружать DHCP сервер для применения конфигурации
```
С помошью 2 команд:
```  
systemctl stop isc-dhcp-server  
systemctl start isc-dhcp-server  
```  
```systemctl enable isc-dhcp-server```  
### IPv6
```nano /etc/dhcp/dhcpd6.conf```
Пример DHCP для IPv6:  
```  
default-lease-time 2592000;  
preferred-lifetime 604800;  
option dhcp-renewal-time 3600;  
option dhcp-rebinding-time 7200;  
allow leasequery;  
  
option dhcp6.info -refresh-time 21600;  
authoritative;  
  
subnet6 2001::1:0/122 {
range6 2001::1:0 2001::1:3e;  
option dhcp6.name-servers 2001::1:2;  
option dhcp6.domain-search "hq.work";
}  
```  
``` apt install radvd ``` 
``` nano /etc/radvd.conf ```
Пример конфигурации Radvd:  
```
interface ens224
{  
MinRtrAdvInterval 3;  
MaxRtrAdvInterval 60;  
AdvSendAdvert on;  
};  
```
Где:  
interface — это имя интерфейса направленного в локальную сеть  
Min и MAX интервалы — это интервалы рассылки объявлений  
AdvSendAdvert — это разрешение на выдачу объявлений от маршрутизатор клиентам  
```  
systemctl stop radvd  
systemctl start radvd  
systemctl enable radvd  
```
# 4 ALL (local)  
|User|Password|Name|  
|---|---|---|  
|Admin|P@ssw0rd|CLI HQ-SRV HQ-R| 
|Branch admin|P@ssw0rd|BR-SRV BR-R| 
|Network admin|P@ssw0rd|HQ-R BR-R BRSRV| 
