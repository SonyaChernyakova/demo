# demo
![image](https://github.com/user-attachments/assets/5ccb31a6-f690-4789-8037-62b7049578a1)
![image](https://github.com/user-attachments/assets/78b44241-aac2-4fd0-8613-39edf773cb04)
HQ-RTR | BR-RTR:  
в режиме глобальной конфигурации hostname {hq-rtr, br-rtr}    
HQ-SRV | HQ-CLI | BR-SRV:  
hostnamectl set-hostname {hq-srv, hq-cli, br-srv}.au-team.irpo; exec bash 

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

HQ-RTR - ip route 0.0.0.0/0 172.16.4.14  
BR-RTR - ip route 0.0.0.0/0 172.16.5.14  
o Интерфейс, к которому подключен HQ-RTR, подключен к сети 172.16.4.0/28  
Настройка производится на EcoRouter:  
en  
conf t  
int ISP  
ip add 172.16.4.1/28  
port te0  
service-instance toISP  
encapsulation untagged  
connect ip interface ISP  
wr  mem  
o Интерфейс, к которому подключен BR-RTR, подключен к сети 172.16.5.0/28  
Настройка производится на EcoRouter:  
en  
conf t  
int ISP  
ip add 172.16.5.1/28  
port te0  
service-instance toISP  
encapsulation untagged  
connect ip interface ISP  
wr  mem  
