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

- Di chuyển vào folder devstack, tạo file local.conf bằng các lệnh:
```
cd devstack/
nano local.conf
```
![image](https://user-images.githubusercontent.com/41882267/90045902-e2e75400-dcf9-11ea-88d0-a5d91f85c2f4.png)

- File local.conf có nội dung như sau. Lưu ý, Host_IP là địa chỉ ip gắn trên card host-only
![image](https://user-images.githubusercontent.com/41882267/90046059-1cb85a80-dcfa-11ea-8a37-b58be025ea4d.png)

- Để bắt đầu cài đặt, sử dụng lệnh:
```
./stack.sh
```

- Ở bước cài đặt nếu có lỗi "Please use apt-cdrom to make this CD-ROM recognized by APT. apt-get update cannot be used to add new CD-ROMs" thì có thể sửa bằng cách 
sử dụng lệnh: 
```
sudo nano -c /etc/apt/sources.list
```
Sau đó comment dòng số 5 và lưu lại.
![image](https://user-images.githubusercontent.com/41882267/90046128-3b1e5600-dcfa-11ea-85a7-ad65d2b7414d.png)

- Sau khi cài đặt xong, trên terminal sẽ có thông báo như sau:
![image](https://user-images.githubusercontent.com/41882267/90046341-89cbf000-dcfa-11ea-96ec-57a855573580.png)

- Chuyển qua chế độ root, để thêm bridge, sử dụng lệnh:
```
sudo su
ovs-vsctl ađ-br br-providernet
```
![image](https://user-images.githubusercontent.com/41882267/90047381-06130300-dcfc-11ea-8d01-564fd73ab50e.png)

- Restart để brigde vừa thêm hoạt động bằng lệnh:
```
systemctl restart devstack@q-*
```
![image](https://user-images.githubusercontent.com/41882267/90047515-3b1f5580-dcfc-11ea-9ece-6a83dc18a0f9.png)

- Sửa file ml2_conf.ini bằng lệnh:
```
nano /etc/neutron/plugins/ml2/ml2_conf.ini
```
![image](https://user-images.githubusercontent.com/41882267/90047704-8c2f4980-dcfc-11ea-8d1c-f1ca0df1bc3b.png)

- Bên trong file ml2_conf.ini, để khai báo physical network, làm như sau:
![image](https://user-images.githubusercontent.com/41882267/90047840-c698e680-dcfc-11ea-96ae-b495feb9d1b1.png)

- Bên trong file ml2_conf.ini, để nối physical network với bridge, làm như sau:
![image](https://user-images.githubusercontent.com/41882267/90047916-e5977880-dcfc-11ea-8682-1cbd066fcf85.png)

sau đó lưu lại. 
- Để gắn card NAT vào bridge br-providernet, sử dụng lệnh:
```
ovs-vsctl add-port br-providernet ens33
```
![image](https://user-images.githubusercontent.com/41882267/90048348-969e1300-dcfd-11ea-8c29-a6abf2068078.png)

- Gỡ bỏ địa chỉ ip trên card ens33 bằng lệnh:
```
ifconfig ens33 0
```
![image](https://user-images.githubusercontent.com/41882267/90048456-bfbea380-dcfd-11ea-92a1-6be37eb251aa.png)

- Để set tĩnh địa chỉ ip cho các interface/bridge, sử dụng lệnh:
```
nano /etc/netplan/01-netcfg.yaml
```
- Sửa file 01-netcfg.yaml như sau:
![image](https://user-images.githubusercontent.com/41882267/90048676-090ef300-dcfe-11ea-9280-367bfe4092c1.png)

- Mở trình duyệt, truy cập vào địa chỉ 172.16.41.128 (địa chỉ host OpenStack) và đăng nhập với username: admin và password: 123456. Chuyển qua project là admin. Để tạo private network cùng với subnet, truy cập theo Project > Network > Networks > Create Network và thiết lập như sau:
![image](https://user-images.githubusercontent.com/41882267/90049742-74a59000-dcff-11ea-8b84-9695f410807f.png)
![image](https://user-images.githubusercontent.com/41882267/90049825-9737a900-dcff-11ea-9d6f-f8b487ab2201.png)
![image](https://user-images.githubusercontent.com/41882267/90049851-a3236b00-dcff-11ea-8e99-a84e6eaeae5e.png)

- Để tạo provider network cùng với subnet, truy cập theo Admin > Network > Networks > Create Network và thiết lập như sau:
![image](https://user-images.githubusercontent.com/41882267/90049990-ccdc9200-dcff-11ea-87d5-94aa71ff39ce.png)
![image](https://user-images.githubusercontent.com/41882267/90050003-d0701900-dcff-11ea-904c-c5b8be161196.png)
![image](https://user-images.githubusercontent.com/41882267/90050015-d403a000-dcff-11ea-97bc-df086703cd4d.png)

- Để tạo router, truy cập vào Project > Network > Routers > Create Router và thiết lập như sau:
![image](https://user-images.githubusercontent.com/41882267/90050195-0f9e6a00-dd00-11ea-9e66-3e037bb42d51.png)

- Để thêm interface của private network vào router Lab vừa tạo, chọn router Lab, chuyển qua tab Interfaces, chọn Add Interface và thiết lập như sau:
![image](https://user-images.githubusercontent.com/41882267/90050394-512f1500-dd00-11ea-8868-ce873005f252.png)

- Để tạo keypair, vào Project > Compute > Key Pairs > Create Key Pair
![image](https://user-images.githubusercontent.com/41882267/90050548-8cc9df00-dd00-11ea-8186-599c0e0f7f9f.png)


