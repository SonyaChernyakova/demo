# demo
```
HQ-RTR	 172.16.40.1/28 	172.16.40.14
BR-RTR 	172.16.50.1/28  	172.16.50.14
HQ-SRV 	192.168.0.61/27 	192.168.0.62
HQ-CLI 	192.168.1.65/27(DHCP) 	192.168.1.78
BR-SRV 	192.168.2.2/28  	192.168.2.1
```
```
Имя Устройства	IPv4	Интерфейс	NIC	Шлюз
ISP	NAT (inet)	ens3	Internet	
	172.16.40.14/28	ens4	ISP_HQ	
	172.16.50.14/28	ens5	ISP_BR	
HQ-RTR	172.16.40.1/28	te0	ISP_HQ	172.16.40.14
	192.168.0.81/29	te1	HQ_NET	
	192.168.0.62/27	te1.100		
	192.168.1.78/27	te1.200		
	172.16.0.1/30	GRE	TUN	
HQ-SW	192.168.0.82/29	ens3	HQ_NET	
	-	ens4	SRV_NET	
	-	ens5	CLI_NET	
HQ-SRV	192.168.0.61/27	ens3	SRV_NET	192.168.0.62
HQ-CLI	192.168.1.65/27(DHCP)	ens3	CLI_NET	192.168.1.78
BR-RTR	172.16.50.1/28	te0	ISP_BR	172.16.50.14
	192.168.2.1/28	te1	BR_NET	
	172.16.0.2/30	GRE	TUN	
BR-SRV	192.168.2.2/28	ens3	BR_NET	192.168.2.1

```
На ISP настройте динамическую сетевую трансляцию в сторону HQ-RTR и BR-RTR  
для доступа к сети Интернет:  
echo net.ipv4.ip_forward=1 > /etc/sysctl.conf
dnf install iptables-services –y   
systemctl enable ––now iptables  
iptables –t nat –A POSTROUTING –s 172.16.40.0/28 –o ens3 –j MASQUERADE  
iptables –t nat –A POSTROUTING –s 172.16.50.0/28 –o ens3 –j MASQUERADE  
iptables-save > /etc/sysconfig/iptables  
systemctl restart iptables  
iptables –L –t nat - должны высветится в Chain POSTROUTING две настроенные подсети.  


HQ-RTR | BR-RTR:  
в режиме глобальной конфигурации hostname {hq-rtr, br-rtr}    
HQ-SRV | HQ-CLI | BR-SRV:  
hostnamectl set-hostname {hq-srv, hq-cli, br-srv}.au-team.irpo; exec bash 



РОУТЕРЫ 

HQ-RTR - ip route 0.0.0.0/0 172.16.40.14 
en  
conf t      
int ISP       
ip add 172.16.40.1/28       
port te0       
service-instance toISP       
encapsulation untagged       
connect ip interface ISP       
wr  mem       


BR-RTR - ip route 0.0.0.0/0 172.16.50.14 
en  
conf t  
int ISP  
ip add 172.16.50.1/28  
port te0  
service-instance toISP  
encapsulation untagged  
connect ip interface ISP  
wr mem

```
Int SRV     
ip add 192.168.2.1/28   
port te1  
service-instance toSRV  
encapsulation untagged    
connect ip interface SRV   
wr  mem 
```

Настройка производится на EcoRouter HQ-RTR: 
```
ip nat pool nat1 192.168.0.1-192.168.0.100 (254)
ip nat pool nat2 192.168.1.65-192.168.1.79 
ip nat source dynamic inside-to-outside pool nat1 overload interface ISP 
ip nat source dynamic inside-to-outside pool nat2 overload interface ISP 

en
conf t
int ISP
ip nat outside
int vl999
ip nat inside
int te1.100
ip nat inside
int te1.200
ip nat inside
```

Настройка производится на EcoRouter BR-RTR: 
```
ip nat pool nat3 192.168.2.2-192.168.2.31  
ip nat source dynamic inside-to-outside pool nat3 overload interface ISP 

en
conf t
int ISP
ip nat outsidе
int SRV
ip nat inside
```



Настройка производится на EcoRouter:  
username net_admin  
password P@$$word  
role admin  


ДЛЯ SW
HQ-RTR

```
port te1  
Service-instance toSW  
Encapsulation untagged
int vl999  
ip add 192.168.0.81/29  
description toSW   
connect port te1 service-instance toSW  
end wr mem

int te1.100  
ip add 192.168.0.62/27  
port te1  
service-instance te1.100  
encapsulation dot1q 100  
rewrite pop 1  
connect ip interface te1.100  

int te1.200  
ip add 192.168.1.78/27  
port te1  
service-instance te1.200  
encapsulation dot1q 200  
rewrite pop 1  
connect ip interface te1.200  
```

HQ SW

nmtui hq 192.168.0.82/29
шлюз 192.168.0.81
-y
systemctl enable --now openvswitch

ovs-vsctl add-br ovs0  
ovs-vsctl add-port ovs0 ens3  
ovs-vsctl set port ens3 vlan_mode=native-untagged tag=999 trunks=999,100,200  
ovs-vsctl add-port ovs0 ovs0-vlan999 tag=999 -- set Interface ovs0-vlan999 type=internal  
ifconfig ovs0-vlan999 inet 192.168.0.82/29 up  

ovs-vsctl add-port ovs0 ens4  
ovs-vsctl set port ens4 tag=100 trunks=100  
ovs-vsctl add-port ovs0 ovs0-vlan100 tag=100 -- set Interface ovs0-vlan100 type=internal  
ifconfig ovs0-vlan100 up  

ovs-vsctl add-port ovs0 ens5  
ovs-vsctl set port ens5 tag=200 trunks=200  
ovs-vsctl add-port ovs0 ovs0-vlan200 tag=200 -- set Interface ovs0-vlan200 type=internal  
ifconfig ovs0-vlan200 up 



Настройка производится на HQ-SRV: 192.168.0.2/26 - 192.168.0.62  

Настройка производится на BR-SRV: 192.168.1.2/27 - 192.168.1.1

useradd -m -u 1015 sshuser  
echo "sshuser:P@ssw0rd" | sudo chpasswd  
usermod -aG wheel sshuser  
nano /etc/sudoers  
sshuser ALL=(ALL) NOPASSWD:ALL


Перед настройкой выполните команду setenforce 0, далее переводим selinux в состояние  
permissive в файле /etc/selinux/config
```
dnf install openssh -y
systemctl enable --now sshd
nano /etc/ssh/sshd_config
Меняем порт на 2024

отступ 3 строки

AllowUsers sshuser
MaxAuthTries 2
Banner /etc/ssh/banner

в /etc/ssh/banner  «Authorized access only»
systemctl restart sshd

ssh sshuser@192.168.1.2 -p 2024 
hq 192.168.0.2
```
ТУНЕЛЬ

```
HQ-RTR:

Interface tunnel.1  
Ip add 172.16.0.1/30  
Ip mtu 1476
ip ospf network broadcast
ip ospf mtu-ignore
Ip tunnel 172.16.4.1 172.16.5.1 mode gre
end

Conf t
Router ospf 1
Ospf router-id  172.16.0.1
network 172.16.0.0 0.0.0.3 area 0
network 192.168.0.0 0.0.0.63 area 0
network 192.168.1.78 0.0.0.15 area 0
passive-interface default
no passive-interface tunnel.1

 router ospf 1
 area 0 authentication 
 ex
 interface tunnel.1  
 ip ospf authentication-key ecorouter  
 wr mem  

BR-RTR:

Interface tunnel.1
Ip add 172.16.0.2/30
Ip mtu 1476
ip ospf mtu-ignore
ip ospf network broadcast
Ip tunnel 172.16.5.1 172.16.4.1 mode gre
end

Conf t
Router ospf 1
Ospf router-id 172.16.0.2
Network 172.16.0.0 0.0.0.3 area 0
Network 192.168.1.0 0.0.0.31 area 0
Passive-interface default
no passive-interface tunnel.1

 router ospf 1  
 area 0 authentication 
 ex  
 interface tunnel.1  
 ip ospf authentication-key ecorouter  
 wr mem  

```
HQ-RTR:
```
ip pool dhcpHQ 192.168.1.65-192.168.1.79
en
conf t
dhcp-server 1
pool dhcpHQ 1
domain-name au-team.irpo
mask 255.255.255.240  
gateway 192.168.1.78  
dns 192.168.0.1  
end


wr mem  
interface te1.200
dhcp-server 1
```
Настройка клиента 
Адрес DNS-сервера для машины HQ-CLI – адрес сервера HQ-SRV

```
hq-srv:
timedatectl set-timezone Europe/Moscow  

hq-rtr: ntp server 172.16.40.1 
ntp timezone UTC+3 
```
```
dnf install bind -y
systemctl enable --now named
nano /etc/named.conf
```
![image](https://github.com/user-attachments/assets/62e3b167-9ba2-47e2-87d1-ae693eb4d068)
![image](https://github.com/user-attachments/assets/18ec9b4f-b969-4abf-911e-f41f9b2a77f2)
```

```
На HQ-CLI и HQ-RTR установить DNS-сервер: 192.168.0.61
```
/var/named/master/au-team
```
```
$TTL    604800  ;
@       IN      SOA     hq-srv.au-team.irpo     root.au-team.irpo       (
                1       ;       Serial
                3600    ;       Refresh
                900     ;       Retry
                1w      ;       Expire
                3600    ;       Minimum TTL
                )
        IN      NS      127.0.0.1.
hq-srv  IN      A       192.168.0.61
hq-rtr  IN      A       192.168.0.62
br-rtr  IN      A       172.16.5.1
hq-cli  IN      A       192.168.0.65
br-srv  IN      A       192.168.1.2

moodle  IN      CNAME   hq-rtr.au-team.irpo.
wiki    IN      CNAME   hq-rtr.au-team.irpo.
```
/var/named/master/168.192.zone 
```
$TTL    3600    ;
@       IN      SOA     hq-srv.au-team.irpo     root.au-team.irpo       (
                1       ;       Serial
                3600    ;       Refresh
                900     ;       Retry
                1w      ;       Expire
                360     ;       Minimum TTL
                )
@       IN      NS      localhost.
2.0     IN      PTR     hq-srv
62.0    IN      PTR     hq-rtr
65.1    IN      PTR     hq-cil
```








