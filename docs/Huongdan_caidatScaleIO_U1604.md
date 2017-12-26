## Hướng dẫn cài đặt ScaleIO và tích hợp với OpenStack

### Mục tiêu LAB
- Mô hình này sử dụng 4 server, trong đó:
  - Host ScaleIO1 cài đặt Gateway, MDM, SDS
  - Host ScaleIO2 cài đặt MDM, SDS
  - Host ScaleIO3 cài đặt TieBreaker, SDS
  - Host Windows để cài đặt, quản trị trên giao diện
- LAB này chỉ phù hợp với việc nghiên cứu các tính năng và demo thử nghiệm, không áp dụng được trong thực tế.

## Mô hình 
- Sử dụng mô hình dưới để cài đặt
![img](../images/openstack-scaleio-network-layout.jpg)

## IP Planning
- Phân hoạch IP cho các máy chủ trong mô hình trên
![img](../images/openstack-scaleio-ip-planning.jpg)

## Chuẩn bị và môi trường LAB
  - Ubuntu Server 16.04 - 64 bit
  - Windows 7 trở lên.
  - Ram 3GB trở lên
  - Ổ cứng từ 90GB trở lên
  - Hệ thống OpenStack Newton được cài đặt sẵn theo hướng dẫn ở [đây](https://github.com/congto/OpenStack-Newton-Scripts)
	
## 1. Cài đặt ScaleIO

### Download bộ cài
  - Download bộ cài ở link sau và giải nén: 
	https://downloads.emc.com/emc-com/usa/ScaleIO/ScaleIO_2.0.1.4_Complete_Linux_SW_Download.zip
	Kết quả:

![img](../images/25.jpg)

### Thực hiện trên tất cả các host
  - Enable root login qua ssh
	```sh
	sudo passwd
	sudo sed -i 's/prohibit-password/yes/' /etc/ssh/sshd_config
	sudo service ssh reload
	```

  - Cài đặt các dependency:
	```sh
	apt-get install libaio1 -y
	```

### Thực hiện trên host ScaleIO1
  - Cài đặt Java 8 runtime
	```sh
	apt-get install openjdk-8-jre binutils -y
	```

	- Cài đặt Gateway, sử dụng file emc-scaleio-gateway_2.0-12000.122_amd64.deb download ở trên, khai báo password tùy ý, password này sẽ sử dụng để log in vào giao diện quản trị ScaleIO
	```sh
	sudo GATEWAY_ADMIN_PASSWORD=[password] dpkg -i emc-scaleio-gateway_2.0-14000.231_amd64.deb
	```

	- Sau khi cài đặt xong, truy cập vào giao diện quản trị theo địa chỉ
	https://172.16.68.80
	Kết quả:

![img](../images/1.jpg)

  - Login vào bằng tk admin, password tạo ở trên, tiếp tục chọn "Get Started"

![img](../images/2.jpg)

  - Lựa chọn "Browser", chọn tất cả các file trong thư mục "ScaleIO_2.0.1.4_UBUNTU_16.04_Download" và tiến hành Upload
![img](../images/3.jpg)

![img](../images/4.jpg)

![img](../images/5.jpg)

  Kết quả:
![img](../images/6.jpg)

  - Ở bước Install, chọn cài đặt mô hình 3 node
![img](../images/7.jpg)

  - Khai báo password của MDM, LIA, accept license, khai báo IP các host ScaleIO và password để ssh vào từng host
![img](../images/8.jpg)

	- Lần lượt thực hiện từng bước cài đặt theo hướng dẫn
![img](../images/9.jpg)

![img](../images/10.jpg)

![img](../images/11.jpg)

  Kết quả:
![img](../images/12.jpg)

### Để quản trị ScaleIO cluster, ScaleIO cung cấp công cụ là ScaleIO GUI, hỗ trợ Windows. Máy Windows này cần kết nối được tới IP quản trị của ScaleIO (172.16.68.80)

  - Cài đặt Java Runtime 1.8

![img](../images/13.jpg)

  - Cài đặt GUI package

![img](../images/14.jpg)

  - Sau khi cài đặt xong, chạy ứng dụng và khai báo IP quản trị của ScaleIO cluster

![img](../images/15.jpg)

![img](../images/16.jpg)

  Giao diện trạng thái của Cluster

![img](../images/17.jpg)

  - Vào tab Backend, trong đây đã hiển thị các host ScaleIO trong Cluster

![img](../images/18.jpg)

  - Chuột phải vào một host, chọn "Add device"

![img](../images/19.jpg)

  - Khai báo label của ổ đĩa trong host, có thể xem bằng lệnh `lsblk` trên host

![img](../images/20.jpg)

  - Thực hiện lần lượt trên từng host, kết quả

![img](../images/21.jpg)

![img](../images/22.jpg)


## 2. Tích hợp OpenStack và ScaleIO

### Trên host OpenStack controller và compute
  - Cài đặt ScaleIO SDC, sử dụng file "EMC-ScaleIO-sdc-2.0-14000.231.Ubuntu.16.04.x86_64.tar" đã download
	```sh
	tar -xvf EMC-ScaleIO-sdc-2.0-14000.231.Ubuntu.16.04.x86_64.tar
	./siob_extract EMC-ScaleIO-sdc-2.0-14000.231.Ubuntu.16.04.x86_64.siob
	```

  - Khởi chạy SDC, khai báo IP của các host MDM
	```sh
	MDM_IP=172.16.68.80,172.16.68.81 dpkg -i EMC-ScaleIO-sdc-2.0-14000.231.Ubuntu.16.04.x86_64.deb
	```

  Nếu sau khi khởi chạy, gặp lỗi sau
  ```sh
	Job for scini.service failed because the control process exited with error code. See "systemctl status scini.service" and "journalctl -xe" for details.
	Job for scini.service failed because the control process exited with error code. See "systemctl status scini.service" and "journalctl -xe" for details.
	Error loading driver, the changes will take place on next reboot
	```

  - Kiểm tra service scini

	```sh
	systemctl status scini.service
	```

  Kết quả:

	```sh
	● scini.service - LSB: This script is responsible to control the scini service.
	   Loaded: loaded (/etc/init.d/scini; bad; vendor preset: enabled)
	   Active: failed (Result: exit-code) since Mon 2017-12-25 18:20:32 UTC; 17s ago
	     Docs: man:systemd-sysv-generator(8)
	Dec 25 18:20:32 compute1 scini[3045]: Error: Unable to fetch scini.ko. Will try to load the existing module...
	Dec 25 18:20:32 compute1 scini[3045]: insmod: ERROR: could not insert module /bin/emc/scaleio/scini.ko: Invalid module format
	Dec 25 18:20:32 compute1 scini[3045]: Error (1) loading module binary /bin/emc/scaleio/scini.ko
	Dec 25 18:20:32 compute1 scini[3045]: rmmod: ERROR: Module scini is not currently loaded
	Dec 25 18:20:32 compute1 scini[3045]: Error (1) unloading module binary /bin/emc/scaleio/scini.ko
	Dec 25 18:20:32 compute1 scini[3045]:  *
	Dec 25 18:20:32 compute1 systemd[1]: scini.service: Control process exited, code=exited status=1
	Dec 25 18:20:32 compute1 systemd[1]: Failed to start LSB: This script is responsible to control the scini service..
	Dec 25 18:20:32 compute1 systemd[1]: scini.service: Unit entered failed state.
	Dec 25 18:20:32 compute1 systemd[1]: scini.service: Failed with result 'exit-code'.
	```

  - Lên trang ftp://ftp.emc.com để down bản scini phù hợp với linux-image của host

	```sh
	wget ftp://QNzgdxXix:Aw3wFAwAq3@ftp.emc.com/Ubuntu/2.0.14000.231/4.4.0-104-generic/scini.tar
	tar -xvf /root/scini.tar
	./siob_extract /root/scini.siob
	mv /root/scini.ko /bin/emc/scaleio
	chmod 777 /bin/emc/scaleio/scini.ko
	```

  - Khởi động lại scini service

	```sh
	systemctl restart scini.service
	systemctl status scini.service
	● scini.service - LSB: This script is responsible to control the scini service.
	   Loaded: loaded (/etc/init.d/scini; bad; vendor preset: enabled)
	   Active: active (exited) since Mon 2017-12-25 18:27:07 UTC; 8min ago
	     Docs: man:systemd-sysv-generator(8)
	  Process: 4617 ExecStart=/etc/init.d/scini start (code=exited, status=0/SUCCESS)
	Dec 25 18:27:01 compute1 scini[4617]: scinia is not ready yet...
	Dec 25 18:27:02 compute1 scini[4617]: scinia is not ready yet...
	Dec 25 18:27:03 compute1 scini[4617]: scinia is not ready yet...
	Dec 25 18:27:04 compute1 scini[4617]: scinia is not ready yet...
	Dec 25 18:27:05 compute1 scini[4617]: scinia is not ready yet...
	Dec 25 18:27:06 compute1 scini[4617]: scinia is not ready yet...
	Dec 25 18:27:07 compute1 scini[4617]: scinia is not ready yet...
	Dec 25 18:27:07 compute1 scini[4617]: Success configuring module
	Dec 25 18:27:07 compute1 scini[4617]:  *
	Dec 25 18:27:07 compute1 systemd[1]: Started LSB: This script is responsible to control the scini service..
	```

  - Kiểm tra trên EMC GUI

![img](../images/24.jpg)	

### Cấu hình cinder-volume trên host Controller

  - Sửa file /etc/cinder/cinder.conf

  ```sh
  [DEFAULT]
	enabled_backends = scaleio
	[scaleio]
	volume_driver = cinder.volume.drivers.emc.scaleio.ScaleIODriver
	volume_backend_name = scaleio
	san_ip = 172.16.68.80
	sio_protection_domain_name = default
	sio_storage_pool_name = default
	sio_storage_pools = default:default
	san_login = admin
	san_password = Sube-1988
	```

  - Tạo type scaleio

	```sh
	cinder type-create scaleio
	cinder type-key scaleio set volume_backend_name=scaleio
	```

### Trên host compute
  - Khởi động lại dịch vụ nova-compute
	```sh
	service nova-compute restart
	```

## 3. Tiến hành tạo và mount volume trên horizon.

Lưu ý: Volume tạo ra trên ScaleIO dung lượng nhỏ nhất là 8GB, ko thay đổi được.

Tham khảo:

[1] - http://node.mu/2017/06/30/scaleio-on-ubuntu-xenial/

[2] -https://www.youtube.com/watch?v=NOnzJeRqaVA

[3] -https://www.emc.com/collateral/technical-documentation/scaleio-quick-start-guide-for-linux.pdf

[4] -https://gist.github.com/clintkitson/92da5036b07a026de10d20ccbf500028

[5] -https://docs.openstack.org/newton/config-reference/block-storage/drivers/emc-scaleio-driver.html