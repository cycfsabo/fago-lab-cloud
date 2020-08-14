# Hướng dẫn cài đặt OpenStack bằng devstack và sử dụng cơ bản Gitlab-CE

## Mục lục


1. [Cài đặt OpenStack](#setup)
    1. Chuẩn bị
    1. Cài đặt
    1. Thiết lập network trên node:
    1. Thiết lập network trên horizon:
1. [Hướng dẫn cài đặt và sử dụng gitlab cơ bản Gitlab-CE](#basic-setup-and-usage-gitlabce)
    1. Dựng instance và cài đặt Gitlab-CE
    2. Hướng dẫn sử dụng cơ bản Gitlab-CE:

## Cài đặt OpenStack <a name="setup"></a>
### Chuẩn bị
Phần mềm VMWare
<br/>
4 cores CPU và 8,5 gb RAM
<br/>
70 gb ổ cứng
<br/>
2 Network Adapter. Trong đó:
<br/>
1 Network Adapter NAT (ens33) dùng để đi ra ngoài internet
<br/>
1 Network Adapter Host-only (Private) (ens34) dùng để host OpenStack
<br/>
Image Ubuntu 64-bit server 18.04.4
<br/>
Sau khi cài đặt xong máy ảo Ubuntu 18.04, bật tất cả network interface.

### Cài đặt
- Thêm user stack 
```
sudo useradd -s /bin/bash -d /opt/stack -m stack
```
- Vì người dùng này sẽ thực hiện nhiều thay đổi đối với hệ thống của bạn, nên người dùng sẽ có các đặc quyền sudo:
```
echo "stack ALL=(ALL) NOPASSWD: ALL" | sudo tee /etc/sudoers.d/stack
sudo su - stack
```
- Download DevStack với --branch stable/(version) để chọn phiên bản:
```
git clone https://git.openstack.org/openstack-dev/devstack --branch stable/train
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

### Thiết lập network trên node:
- Chuyển qua chế độ root, để thêm bridge br-providernet, sử dụng lệnh:
```
sudo su
ovs-vsctl ađ-br br-providernet
```
![image](https://user-images.githubusercontent.com/41882267/90047381-06130300-dcfc-11ea-8d01-564fd73ab50e.png)

- Restart để brigde br-providernet hoạt động bằng lệnh:
```
systemctl restart devstack@q-*
```
![image](https://user-images.githubusercontent.com/41882267/90047515-3b1f5580-dcfc-11ea-9ece-6a83dc18a0f9.png)

- Sửa file ml2_conf.ini bằng lệnh:
```
nano /etc/neutron/plugins/ml2/ml2_conf.ini
```
![image](https://user-images.githubusercontent.com/41882267/90047704-8c2f4980-dcfc-11ea-8d1c-f1ca0df1bc3b.png)

- Bên trong file ml2_conf.ini, khai báo physical network providernet như sau:

![image](https://user-images.githubusercontent.com/41882267/90047840-c698e680-dcfc-11ea-96ae-b495feb9d1b1.png)

- Bên trong file ml2_conf.ini, để nối physical network providernet với bridge br-providernet, làm như sau:

![image](https://user-images.githubusercontent.com/41882267/90047916-e5977880-dcfc-11ea-8682-1cbd066fcf85.png)

sau đó lưu lại. 
- Để gắn interface NAT vào bridge br-providernet, sử dụng lệnh:
```
ovs-vsctl add-port br-providernet ens33
```
![image](https://user-images.githubusercontent.com/41882267/90048348-969e1300-dcfd-11ea-8c29-a6abf2068078.png)

- Gỡ bỏ địa chỉ ip trên interface ens33 bằng lệnh:
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

- Mở trình duyệt, truy cập vào địa chỉ 172.16.41.128 (địa chỉ host OpenStack) và đăng nhập với username: admin và password: 123456. 
<br/>
Chuyển qua project tên admin.
<br/>
- Để tạo keypair, vào Project > Compute > Key Pairs > Create Key Pair. Sau khi tạo xong ssh key, trình duyệt sẽ tự động tải về file key dùng để ssh.

![image](https://user-images.githubusercontent.com/41882267/90172757-a6852800-ddcd-11ea-91e3-3ad569e15d15.png)

- Để tạo flavor, vào Admin > Compute > Flavors > Create Flavor và thiết lập như sau:

![image](https://user-images.githubusercontent.com/41882267/90095347-941dd680-dd5a-11ea-9ea7-4cac89d80e9b.png)

- Để tạo Image, vào Admin > Compute > Images > Create Image và thiết lập như sau. Lưu ý sử dụng cloud image như trên trang chủ khuyến cáo.

![image](https://user-images.githubusercontent.com/41882267/90095583-49e92500-dd5b-11ea-9e89-754ab52ccf9e.png)

### Thiết lập network trên horizon: 
Ở đây chúng ta sử dụng mô hình network provider.
<br/>
Để tạo private network cùng với subnet, truy cập theo Project > Network > Networks > Create Network và thiết lập như sau:

![image](https://user-images.githubusercontent.com/41882267/90049742-74a59000-dcff-11ea-8b84-9695f410807f.png)

![image](https://user-images.githubusercontent.com/41882267/90049825-9737a900-dcff-11ea-9d6f-f8b487ab2201.png)

![image](https://user-images.githubusercontent.com/41882267/90049851-a3236b00-dcff-11ea-8e99-a84e6eaeae5e.png)

- Để tạo provider network cùng với subnet, truy cập theo Admin > Network > Networks > Create Network và thiết lập như sau:

![image](https://user-images.githubusercontent.com/41882267/90172304-029b7c80-ddcd-11ea-8ead-f50cf98e0218.png)

![image](https://user-images.githubusercontent.com/41882267/90050003-d0701900-dcff-11ea-904c-c5b8be161196.png)

![image](https://user-images.githubusercontent.com/41882267/90050015-d403a000-dcff-11ea-97bc-df086703cd4d.png)

- Để tạo router, truy cập vào Project > Network > Routers > Create Router và thiết lập như sau:

![image](https://user-images.githubusercontent.com/41882267/90172688-940aee80-ddcd-11ea-8277-bb7066892809.png)

- Để thêm interface của private network vào router Lab vừa tạo, chọn router Lab, chuyển qua tab Interfaces, chọn Add Interface và thiết lập như sau:

![image](https://user-images.githubusercontent.com/41882267/90172819-bef54280-ddcd-11ea-9df2-61ff358ddd1d.png)

- Truy cập Project > Network > Security Groups, chọn Manage Rules security group default và thiết lập như sau:

Thêm rule để có thể ping instance:

![image](https://user-images.githubusercontent.com/41882267/90214971-7452e500-de24-11ea-9ae0-81d33fbb31ff.png)

Thêm rule để có thể ssh instance:

![image](https://user-images.githubusercontent.com/41882267/90214987-7f0d7a00-de24-11ea-80db-0c7054a1f04d.png)

Thêm rule để có thể truy cập web host bởi instance:

![image](https://user-images.githubusercontent.com/41882267/90214998-86348800-de24-11ea-94d9-1e4bf2fa4eb7.png)

![image](https://user-images.githubusercontent.com/41882267/90215013-9482a400-de24-11ea-8e62-4253bd2a6eb8.png)

Kết quả:

![image](https://user-images.githubusercontent.com/41882267/90214948-5d13f780-de24-11ea-843a-b18b7de3400b.png)


## Hướng dẫn cài đặt và sử dụng cơ bản Gitlab-CE <a name="basic-setup-and-usage-gitlabce"></a>
### Dựng instance và cài đặt Gitlab-CE

- Để tạo Instance, vào Project > Compute > Instances > Launch Instance

![image](https://user-images.githubusercontent.com/41882267/90098694-04305a80-dd63-11ea-946a-916566d4fe25.png)

![image](https://user-images.githubusercontent.com/41882267/90098744-1f02cf00-dd63-11ea-9f93-c3bccad9f0fb.png)

"test2" là flavor tự tạo có các giá trị phù hơp để instance có thể chạy Gitlab-CE

![image](https://user-images.githubusercontent.com/41882267/90098775-2b872780-dd63-11ea-9806-829fd85d10cb.png)

![image](https://user-images.githubusercontent.com/41882267/90098791-393cad00-dd63-11ea-8e63-0ac399aa5069.png)

![image](https://user-images.githubusercontent.com/41882267/90098935-96386300-dd63-11ea-82d9-689dc3dcf74e.png)

![image](https://user-images.githubusercontent.com/41882267/90098971-a3ede880-dd63-11ea-80c6-34cc6d1e7959.png)

- Để instance có thể truy cập được từ network provider, cần phải gán cho nó 1 floating ip. Để gán floating ip cho một instance, ta làm như sau:

![image](https://user-images.githubusercontent.com/41882267/90119356-a52f0d80-dd83-11ea-971f-741f58bac94c.png)

- Chọn biểu tượng dấu "+" để tạo mới floating ip:

![image](https://user-images.githubusercontent.com/41882267/90119403-b9730a80-dd83-11ea-8b63-7b05f23881dc.png)

- Ở đây chọn provider-net, sau đó chọn Allocate IP:

![image](https://user-images.githubusercontent.com/41882267/90119593-f808c500-dd83-11ea-9d28-310ce595818e.png)

- Sau đó chọn Associate

![image](https://user-images.githubusercontent.com/41882267/90119768-30a89e80-dd84-11ea-85e0-534c925c5abb.png)

- Sau khi instance hoàn tất cài đặt, ssh vào instance từ laptop bằng lệnh:
```
ssh -i <path/to/key> ubuntu@<floatingip>
```
![image](https://user-images.githubusercontent.com/41882267/90132696-796a5280-dd98-11ea-93bc-f5f49bc4d4e7.png)

- Update Ubuntu bằng lệnh:
```
apt-get update
```

- Install ssh và postfix:
```
apt-get install openssh-server postfix -y
```
![image](https://user-images.githubusercontent.com/41882267/90136771-e2ed5f80-dd9e-11ea-9095-9b69001fd89a.png)

- Chọn OK:

![image](https://user-images.githubusercontent.com/41882267/90134157-ee3e8c00-dd9a-11ea-9477-c6dfaf54cc99.png)


- Chọn Internet Site:

![image](https://user-images.githubusercontent.com/41882267/90173882-4a230800-ddcf-11ea-836b-fda01343105d.png)

- System mail name là www.labcloud.com:

![image](https://user-images.githubusercontent.com/41882267/90173933-632bb900-ddcf-11ea-97df-7401f11ac3bf.png)

- Tải về package cài đặt gitlab-ce 10.5.7 bằng lệnh:
```
wget --content-disposition https://packages.gitlab.com/gitlab/gitlab-ce/packages/ubuntu/xenial/gitlab-ce_10.5.7-ce.0_amd64.deb/download.deb
```
![image](https://user-images.githubusercontent.com/41882267/90171540-db907b00-ddcb-11ea-8e08-43dd02c9b57c.png)


- Cài đặt gitlab-ce bằng lệnh:
```
dpkg -i gitlab-ce_10.5.7-ce.0_amd64.deb
```
![image](https://user-images.githubusercontent.com/41882267/90134241-129a6880-dd9b-11ea-81d3-aed853abf598.png)

- Sửa file gitlab.rb bằng lệnh:
```
nano -c /etc/gitlab/gitlab.rb
```
Sửa dòng 13:
<br/>
![image](https://user-images.githubusercontent.com/41882267/90141081-d79d3280-dda4-11ea-8041-270c1b3d1903.png)

Sửa dòng 451:
<br/>
![image](https://user-images.githubusercontent.com/41882267/90141137-ea176c00-dda4-11ea-8511-ad93701c1bf5.png)

Sửa dòng 1085:
<br/>
![image](https://user-images.githubusercontent.com/41882267/90141237-10d5a280-dda5-11ea-9b6f-522b66389942.png)

- Sau đó reconfig gitlab bằng lệnh:
```
gitlab-ctl reconfigure
```
![image](https://user-images.githubusercontent.com/41882267/90141449-55613e00-dda5-11ea-95bc-4d2f43470d65.png)

- Nếu trong quá trình reconfig bị lỗi timeout thì có thể tiếp tục thực hiện lại reconfig để hệ thống xử lý những service còn lại. Để kiểm tra trạng thái các service, sử dụng lệnh:
```
gitlab-ctl status
```
![image](https://user-images.githubusercontent.com/41882267/90141983-05cf4200-dda6-11ea-9664-5b3833c86951.png)

- Từ laptop, mở trình duyệt và nhập vào floating ip của instance. Lần đầu sẽ phải tạo lại password mới:

![image](https://user-images.githubusercontent.com/41882267/90143020-4aa7a880-dda7-11ea-88f4-88b57d01defd.png)

- Đăng nhập với username là root, password là password vừa tạo:

![image](https://user-images.githubusercontent.com/41882267/90143168-7a56b080-dda7-11ea-9251-9e0d09484a4a.png)

- Kết quả khi đăng nhập thành công:

![image](https://user-images.githubusercontent.com/41882267/90143238-90647100-dda7-11ea-8598-b8086c4a16b0.png)

### Hướng dẫn sử dụng cơ bản Gitlab-CE:

- Kiểm tra version git:
```
git --version
```
![image](https://user-images.githubusercontent.com/41882267/90227850-6d859b80-de3f-11ea-9d56-cb2061098865.png)

- Thêm Git username "hungch" bằng lệnh:
``
git config --global user.name "hungch"
``
-  Kiểm tra username vừa thêm vào bằng lệnh:
```
git config --global user.name
```
![image](https://user-images.githubusercontent.com/41882267/90228039-c1908000-de3f-11ea-9702-febfdb4412da.png)

-  Set email "huuhungf@gmail.com" bằng lệnh:
```
git config --global user.email "huuhungf@gmail.com"
```
- Kiểm tra email vừa thêm vào bằng lệnh:
```
git config --global user.email
```
![image](https://user-images.githubusercontent.com/41882267/90228187-fac8f000-de3f-11ea-9dfa-bde55f6dd633.png)

- Kiểm tra các thông tin đã được thêm vào bằng lệnh:
```
git config --global --list
```
![image](https://user-images.githubusercontent.com/41882267/90228711-e46f6400-de40-11ea-9b81-4f9d25ab1332.png)

- Mở trình duyệt, chọn Create a project và thiết lập như sau:

![image](https://user-images.githubusercontent.com/41882267/90229965-fce07e00-de42-11ea-8be4-1b21a47ec712.png)

- Clone project bằng lệnh:
```
git clone http://192.168.182.237/root/example1.git
```
![image](https://user-images.githubusercontent.com/41882267/90230158-4e890880-de43-11ea-931f-0749f1e5a5eb.png)

- Tạo file Readme.md bằng lệnh:
```
cd example1
touch Readme.md
```
- Add file Readme.md bằng lệnh:
```
git add Readme.md
```
- Commit bằng lệnh:
```
git commit -m "add Readme"
```
- Đẩy file Readme.md đên branch master bằng lệnh:
```
git push -u origin master
```
Và nhập username: root/password: 12345678 (do mình set từ lần đầu đăng nhập gitlab-ce)

![image](https://user-images.githubusercontent.com/41882267/90230710-2b128d80-de44-11ea-8134-0bdd6fd19de3.png)

- Thay đổi nội dung file Readme.md trên giao diện web, pull file mới về bằng lệnh:
```
git pull origin master
```
![image](https://user-images.githubusercontent.com/41882267/90230863-64e39400-de44-11ea-83ba-f106c8d2cf15.

- Thay đổi nội dung file Readme.md và tạo thêm file install.sh:
```
nano Readme.md
touch install.sh
```
- Thực hiện các lệnh sau để push:
```
git add *
git commit -m "Update Readme and create install.sh"
git push -u origin master
```
![image](https://user-images.githubusercontent.com/41882267/90233613-9c543f80-de48-11ea-871e-c7b03fa85a00.png)

- Lúc này, trên gitlab đã thay đổi:

![image](https://user-images.githubusercontent.com/41882267/90233680-b4c45a00-de48-11ea-967a-71693a83e000.png)

- Tạo branch mới và chuyển qua branch đó bằng lệnh:
```
git checkout -b hungch
```
![image](https://user-images.githubusercontent.com/41882267/90233951-184e8780-de49-11ea-9f23-815575c41a25.png)

- Thay đổi nội dunng file Readme.md và push lên branch hungch.:
```
nano Readme.md 
git add *
git commit -m "update Readme hungch"
git push -u origin hungch
```
![image](https://user-images.githubusercontent.com/41882267/90234229-8e52ee80-de49-11ea-89be-32e0a42b9c7f.png)

- Truy cập vào đường link "http://192.168.182.237/root/example1/merge_requests/new?merge_request%5Bsource_branch%5D=hungch" dùng để tạo merge request:

![image](https://user-images.githubusercontent.com/41882267/90236079-77fa6200-de4c-11ea-83bf-b2b1f0fd6eff.png)

- Chọn merge để merge request từ branch hungch vào branch master:

![image](https://user-images.githubusercontent.com/41882267/90236296-c1e34800-de4c-11ea-8d43-164e9a48e1f6.png)

- Quay trở lại file để kiểm tra thay đổi:

![image](https://user-images.githubusercontent.com/41882267/90236529-125aa580-de4d-11ea-98f7-a4872238e1c6.png)



