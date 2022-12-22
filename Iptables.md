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
* Xem cấu hình rule vừa tạo
``` 
iptables -L -n -v --line-number
```
![image](https://user-images.githubusercontent.com/91528234/209088589-4935c033-0ce5-49b3-ae87-39441168f9a6.png)
## Kết quả 
* Trước khi cấu hình tường lửa
![image](https://user-images.githubusercontent.com/91528234/209089445-76dd1bd0-56b9-4ddc-9ebf-f97fa011c786.png)
* ssh được vào server
* Sau khi cấu hình tường lửa
![image](https://user-images.githubusercontent.com/91528234/209089734-aa37a843-5116-4735-bd78-6376223b2116.png)
* Rule cấu hình đã chặn client ssh vào server
# LAB2
## Mô hình
![image](https://user-images.githubusercontent.com/91528234/209097631-95809be2-fa00-4f16-9aca-dd204221db53.png)

## Tạo máy ảo Ubuntu Server 16.04 có ip `172.16.69.11` và ip `10.10.10.11`
![image](https://user-images.githubusercontent.com/91528234/209097811-38badb0e-4e9a-4dcf-af0d-d87445725a08.png)
## Tạo client Ubuntu Desktop 20.04 có ip `10.10.10.11` và ping về server
![image](https://user-images.githubusercontent.com/91528234/209098271-52fff89a-6312-4236-8fe0-4b7f95fcc477.png)
![image](https://user-images.githubusercontent.com/91528234/209098606-bb461349-e6e4-4048-b058-53683ca07687.png)

## Tạo Lan với ip `172.16.69.12` và ping về server
![image](https://user-images.githubusercontent.com/91528234/209098477-8d2865c8-5a91-4ebb-bef6-9d28daa1b45b.png)
![image](https://user-images.githubusercontent.com/91528234/209098665-296ddd4a-0baf-4a6c-ae4c-a0fa1b01d46f.png)
## Cấu hình tường lửa cho Server với quyền root
* Trên Server Kích hoạt iptables fordward packet, cần sửa file `/etc/sysctl.conf`:
```
net.ipv4.ip_forward = 1
```
* Chạy lệnh để kiểm tra cài đặt.
```
sysctl -p /etc/sysctl.conf
```

* Sau đó:
```
/etc/init.d/procps restart
```
* DROP INPUT, ACCEPT OUTPUT và DROP FORWARD:
```
iptables -P INPUT DROP
iptables -P OUTPUT ACCEPT
iptables -P FORWARD DROP
```
* ACCEPT Established Connection.
```
iptables -A FORWARD -m state --state ESTABLISHED,RELATED -j ACCEPT
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
```
* ACCEPT kết nối từ loopback:
```
iptables -A INPUT -s 127.0.0.1 -d 127.0.0.1 -j ACCEPT
```
* ACCEPT kết nối Ping với 5 lần mỗi phút từ mạng LAN.
```
iptables -A INPUT -p icmp --icmp-type echo-request -m limit --limit 5/m --limit-burst 5 -s 10.10.10.0/24 -d 10.10.10.11 -j ACCEPT
```
* ACCEPT kết nối SSH từ trong mạng LAN:
```
iptables -A INPUT -p tcp -s 10.10.10.0/24 -d 10.10.10.11 --dport 22 -m state --state NEW -j ACCEPT
```
* ACCEPT Outgoing gói tin qua Server từ network (10.10.10.0/24) và nat địa chỉ nguồn của gói tin.
```
iptables -A FORWARD -i ens37 -o ens33 -j ACCEPT
iptables -t nat -A POSTROUTING -o ens33 -s 10.10.10.0/24 -j SNAT --to-source 10.10.10.11
```
hoặc
```
iptables -A FORWARD -i ens38 -o ens33 -j ACCEPT
iptables -t nat -A POSTROUTING -o ens33 -s 10.10.10.0/24 -j MASQUERADE
```






