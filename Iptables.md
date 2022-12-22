# Lab1
## Mô hình
![image](https://user-images.githubusercontent.com/91528234/209085075-8fcc5d54-8f8b-4820-8d4f-e4eaeca4bde6.png)

## Tạo máy ảo Ubuntu Server 16.04 có ip `172.16.69.11`

![image](https://user-images.githubusercontent.com/91528234/209084862-139ad3bf-8fd3-4afa-ab1f-edcd7b801fe7.png)
## Tạo client Ubuntu Desktop 20.04 có ip `172.16.69.101`

![image](https://user-images.githubusercontent.com/91528234/209085300-47eef6f6-e311-461f-a89f-bd2f3366542a.png)
## Cấu hình cho network adapter cho client và server cùng một mạng `vmnet1`
![image](https://user-images.githubusercontent.com/91528234/209085592-ee55552e-5da8-46ea-83eb-7180001b7bff.png)
## ping từ client đến server
![image](https://user-images.githubusercontent.com/91528234/209086151-f8a5ad8a-5c2c-40ed-9dc3-b4b3e71fdbd6.png)
## Cấu hình tường lửa cho Server với quyền root
* Tạo default rule DROP INPUT, ACCEPT OUTPUT và DROP FORWARD
```
iptables -P INPUT DROP
iptables -P OUTPUT ACCEPT
iptables -P FORWARD DROP
```
* Xem cấu hình rule
```
iptables -S
```
![image](https://user-images.githubusercontent.com/91528234/209087382-71de3580-21c2-462e-a3f1-9ca88d0ef11a.png)


* Tạo Rule ACCEPT Established Connection.
```
iptables -A INPUT -p tcp -m state --state ESTABLISHED -j ACCEPT
iptables -A INPUT -p udp -m state --state ESTABLISHED -j ACCEPT
```
* Tạo rule ACCEPT kết nối từ card loopback:
```
iptables -A INPUT -s 127.0.0.1 -d 127.0.0.1 -j ACCEPT
```
* Tạo rule ACCEPT kết nối Ping với 5 lần mỗi phút từ mạng LAN.

```
iptables -A INPUT -p icmp --icmp-type echo-request -m limit --limit 5/m --limit-burst 5 -m state --state NEW,ESTABLISHED -s 172.16.69.0/24 -d 172.16.69.11 -j ACCEPT
```
* Tạo rule ACCEPT SSH
```
iptables -A INPUT -p tcp -m state --state NEW -s 172.16.69.0/24 -d 172.16.69.11 --dport 22 -j ACCEPT
```

