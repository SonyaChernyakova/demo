# demo
![image](https://github.com/user-attachments/assets/5ccb31a6-f690-4789-8037-62b7049578a1)
![image](https://github.com/user-attachments/assets/78b44241-aac2-4fd0-8613-39edf773cb04)

На ISP настройте динамическую сетевую трансляцию в сторону HQ-RTR и BR-RTR  
для доступа к сети Интернет:  
echo net.ipv4.ip_forward=1 > /etc/sysctl.conf
dnf install iptables-services –y   
systemctl enable ––now iptables  
iptables –t nat –A POSTROUTING –s 172.16.4.0/28 –o ens3 –j MASQUERADE  
iptables –t nat –A POSTROUTING –s 172.16.5.0/28 –o ens3 –j MASQUERADE  
iptables-save > /etc/sysconfig/iptables  
systemctl restart iptables  
iptables –L –t nat - должны высветится в Chain POSTROUTING две настроенные подсети.  


HQ-RTR | BR-RTR:  
в режиме глобальной конфигурации hostname {hq-rtr, br-rtr}    
HQ-SRV | HQ-CLI | BR-SRV:  
hostnamectl set-hostname {hq-srv, hq-cli, br-srv}.au-team.irpo; exec bash 




РОУТЕРЫ 

HQ-RTR - ip route 0.0.0.0/0 172.16.4.14 
en  
conf t      
int ISP       
ip add 172.16.4.1/28       
port te0       
service-instance toISP       
encapsulation untagged       
connect ip interface ISP       
wr  mem       

BR-RTR - ip route 0.0.0.0/0 172.16.5.14 
en  
conf t  
int ISP  
ip add 172.16.5.1/28  
port te0  
service-instance toISP  
encapsulation untagged  
connect ip interface ISP  
wr mem

Int SRV     
ip add 192.168.1.1/27   |
port te1  |
service-instance toSRV  |
encapsulation untagged   |
int SRV   |
connect port te1 service-instance toSRV   |
wr  mem 


Настройка производится на EcoRouter HQ-RTR: 

ip nat pool nat1 192.168.0.1-192.168.0.254

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


Настройка производится на EcoRouter BR-RTR: 

ip nat pool nat3 192.168.1.2-192.168.1.31  

ip nat source dynamic inside-to-outside pool nat3 overload interface ISP 

en
conf t
int ISP
ip nat outsidе
int SRV
ip nat inside




Настройка производится на EcoRouter:  
username net_admin  
password P@$$word  
role admin  


ДЛЯ SW
HQ-RTR

int vl999  
ip add 192.168.0.81/29  
description toSW  
port te1  
Service-instance toSW  
Encapsulation untagged  
ex  
Int vl999  
connect port te1 service-instance toSW  

int te1.100  
ip add 192.168.0.62/26  
port te1  
service-instance te1.100  
encapsulation dot1q 100  
rewrite pop 1  
connect ip interface te1.100  

int te1.200  
ip add 192.168.1.78/28  
port te1  
service-instance te1.200  
encapsulation dot1q 200  
rewrite pop 1  
connect ip interface te1.200  


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

useradd -m -u 1010 sshuser  
echo "sshuser:P@ssw0rd" | sudo chpasswd  
usermod -aG wheel sshuser  
nano /etc/sudoers  
sshuser ALL=(ALL) NOPASSWD:ALL


Перед настройкой выполните команду setenforce 0, далее переводим selinux в состояние  
permissive в файле /etc/selinux/config

 dnf install openssh - если не установлен
 systemctl enable --now sshd
 nano /etc/ssh/sshd_config
 Меняем порт на 2024
  AllowUsers sshuser
  MaxAuthTries 2
  Banner /etc/ssh/banner 
  в /etc/ssh/banner  
  «Authorized access only»
  systemctl restart sshd
  ssh sshuser@192.168.1.2 -p 2024 
  hq 192.168.0.2

