## Фильтрация трафика - firewalld, iptables
### Цели занятия:
- настраивать файрвалл с использованием iptables/firewalld;
- настраивать NAT;
- пробрасывать порты;
- настраивать взаимодействие с роутингом;
- понимать работу таблиц и цепочек.
____________________

#### iptables inetRouter:
```
*nat
:PREROUTING ACCEPT [1:44]
:INPUT ACCEPT [1:44]
:OUTPUT ACCEPT [0:0]
:POSTROUTING ACCEPT [0:0]
-A POSTROUTING ! -d 192.168.0.0/16 -o eth0 -j MASQUERADE
COMMIT

*filter
:INPUT DROP [0:0]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [0:0]
:TRAFFIC - [0:0]
:SSH-INPUT - [0:0]
:SSH-INPUTTWO - [0:0]

-A INPUT -j TRAFFIC
-A TRAFFIC -p icmp --icmp-type any -j ACCEPT
-A TRAFFIC -m state --state ESTABLISHED,RELATED -j ACCEPT
-A TRAFFIC -m state --state NEW -m tcp -p tcp --dport 22 -m recent --rcheck --seconds 30 --name SSH2 -j ACCEPT
-A TRAFFIC -m state --state NEW -m tcp -p tcp -m recent --name SSH2 --remove -j DROP
-A TRAFFIC -m state --state NEW -m tcp -p tcp --dport 9991 -m recent --rcheck --name SSH1 -j SSH-INPUTTWO
-A TRAFFIC -m state --state NEW -m tcp -p tcp -m recent --name SSH1 --remove -j DROP
-A TRAFFIC -m state --state NEW -m tcp -p tcp --dport 7777 -m recent --rcheck --name SSH0 -j SSH-INPUT
-A TRAFFIC -m state --state NEW -m tcp -p tcp -m recent --name SSH0 --remove -j DROP
-A TRAFFIC -m state --state NEW -m tcp -p tcp --dport 8881 -m recent --name SSH0 --set -j DROP
-A SSH-INPUT -m recent --name SSH1 --set -j DROP
-A SSH-INPUTTWO -m recent --name SSH2 --set -j DROP
-A TRAFFIC -j DROP
COMMIT
```
#### iptables InetRouter2:
```
*nat
:PREROUTING ACCEPT [0:0]
:INPUT ACCEPT [0:0]
:OUTPUT ACCEPT [0:0]
:POSTROUTING ACCEPT [0:0]
-A PREROUTING -p tcp -m tcp --dport 8080 -j DNAT --to-destination 192.168.0.2:80
-A POSTROUTING --dst 192.168.0.2 -p tcp --dport 80 -j SNAT --to-source 192.168.56.11:8080
COMMIT

*filter
:INPUT ACCEPT [0:0]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [0:0]
-A INPUT -i lo -j ACCEPT
COMMIT
```
#### Проверка Port Knocking:
Пароль root@inetRouter: 1234
```
root@centralRouter:~# ssh root@192.168.255.1
ssh: connect to host 192.168.255.1 port 22: Connection timed out
root@centralRouter:~# ./knock.sh 192.168.255.1 8881 7777 9991
Starting Nmap 7.80 ( https://nmap.org ) at 2024-03-28 08:29 UTC
Warning: 192.168.255.1 giving up on port because retransmission cap hit (0).
Nmap scan report for _gateway (192.168.255.1)
Host is up (0.00068s latency).

PORT     STATE    SERVICE
8881/tcp filtered galaxy4d
MAC Address: 08:00:27:CB:0E:3B (Oracle VirtualBox virtual NIC)

Nmap done: 1 IP address (1 host up) scanned in 0.36 seconds
Starting Nmap 7.80 ( https://nmap.org ) at 2024-03-28 08:29 UTC
Warning: 192.168.255.1 giving up on port because retransmission cap hit (0).
Nmap scan report for _gateway (192.168.255.1)
Host is up (0.0019s latency).

PORT     STATE    SERVICE
7777/tcp filtered cbt
MAC Address: 08:00:27:CB:0E:3B (Oracle VirtualBox virtual NIC)

Nmap done: 1 IP address (1 host up) scanned in 0.18 seconds
Starting Nmap 7.80 ( https://nmap.org ) at 2024-03-28 08:29 UTC
Warning: 192.168.255.1 giving up on port because retransmission cap hit (0).
Nmap scan report for _gateway (192.168.255.1)
Host is up (0.00047s latency).

PORT     STATE    SERVICE
9991/tcp filtered issa
MAC Address: 08:00:27:CB:0E:3B (Oracle VirtualBox virtual NIC)

Nmap done: 1 IP address (1 host up) scanned in 0.18 seconds
root@centralRouter:~# ssh root@192.168.255.1
The authenticity of host '192.168.255.1 (192.168.255.1)' can't be established.
ED25519 key fingerprint is SHA256:PYOVL5vY103A2+GK2Z19nB3WJKqntwKBnPsDAIcjlBs.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '192.168.255.1' (ED25519) to the list of known hosts.
root@192.168.255.1's password: 
root@inetRouter:~# 
```
#### Проверка проброса порта:
```
feelins@FeeLinS-PC:~$ curl 192.168.56.11:8080
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
feelins@FeeLinS-PC:~$ 
```