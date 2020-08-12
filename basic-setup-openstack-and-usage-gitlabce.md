# Hướng dẫn cài đặt OpenStack bằng devstack và sử dụng cơ bản Gitlab-CE

## Mục lục


1. [Cài đặt OpenStack](#setup)
1. [Hướng dẫn sử dụng cơ bản Gitlab-CE](#basic-usage)

## Cài đặt OpenStack <a name="setup"></a>

Chuẩn bị phần mềm VMWare
<br/>
4 core CPU, 8,5gb RAM
<br/>
70gb ổ cứng
<br/>
2 Network Adapter. Trong đó:
<br/>
1 card NAT (ens33) dùng để gắn vào network provider
<br/>
1 card Host-only (ens34) dùng để host openstack
<br/>
Image Ubuntu 64-bit server 18.04.4
<br/>

Sau khi cài đặt xong máy ảo Ubuntu 18.04, bật tất cả card mạng lên.


- Thêm user stack 
```
sudo useradd -s /bin/bash -d /opt/stack -m stack
```
- Vì người dùng này sẽ thực hiện nhiều thay đổi đối với hệ thống của bạn, nên người dùng sẽ có các đặc quyền sudo:
```
echo "stack ALL=(ALL) NOPASSWD: ALL" | sudo tee /etc/sudoers.d/stack
sudo su - stack
```
- Download DevStack:
```
git clone https://opendev.org/openstack/devstack
```
![image](https://user-images.githubusercontent.com/41882267/90045688-969c1400-dcf9-11ea-837c-5c1221174686.png)
