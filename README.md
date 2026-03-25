
# 🛡️ Cuckoo Sandbox Setup Guide

## 💻 Yêu cầu hệ thống
- **Host OS:** Ubuntu 18.04 LTS  
- **Guest OS:** Windows 7 Ultimate x64 (ISO)  
- **Phần mềm:** VirtualBox, Python 2.7, MongoDB, TCPDump  

---

## 🛠 Thiết lập môi trường Host (Ubuntu)

### Cập nhật hệ thống & cài đặt dependencies
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y virtualbox python python-pip python-dev \
libffi-dev libssl-dev python-virtualenv python-setuptools \
libjpeg-dev zlib1g-dev swig tcpdump apparmor-utils
````

### Cấu hình quyền cho tcpdump

```bash
sudo aa-disable /usr/sbin/tcpdump
sudo groupadd pcap
sudo usermod -a -G pcap cuckoo
sudo chgrp pcap /usr/sbin/tcpdump
sudo setcap cap_net_raw,cap_net_admin=eip /usr/sbin/tcpdump
```

---

## 🌐 Cấu hình mạng VirtualBox

### Tạo Host-Only Network

```bash
VBoxManage hostonlyif create
VBoxManage hostonlyif ipconfig vboxnet0 --ip 192.168.56.1 --netmask 255.255.255.0
```

---

## 🖥 Thiết lập máy ảo phân tích (Windows 7 Guest)

### Cấu hình cơ bản

* Network: **Host-only Adapter (vboxnet0)**
* IP tĩnh: `192.168.56.101`

### Vô hiệu hóa bảo mật

* Tắt UAC
* Tắt Windows Firewall
* Tắt Windows Update

### Cài đặt Agent

1. Cài Python 2.7 trên Windows
2. Tạo HTTP server trên Ubuntu để transfer file:

```bash
python -m SimpleHTTPServer 8000
```

3. Tải `agent.py` về Windows
4. Copy vào thư mục:

```
C:\Users\<User>\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup
```

5. Đổi tên thành:

```
agent.pyw
```

### Snapshot

* Tạo snapshot khi hệ thống ở trạng thái **clean + agent đang chạy**

---

## ⚙️ Cài đặt và cấu hình Cuckoo Sandbox

### Khởi tạo môi trường

```bash
virtualenv venv
source venv/bin/activate
pip install -U pip setuptools
pip install -U cuckoo
cuckoo init
```

### Cấu hình file quan trọng

* **virtualbox.conf**

  * Tên VM
  * Snapshot
  * Interface: `vboxnet0`

* **cuckoo.conf**

  * IP host: `192.168.56.1`

* **reporting.conf**

  * Enable HTML report
  * Enable MongoDB

---

## 🔍 Phân tích mã độc

### Bước 1: Khởi chạy Cuckoo Core

```bash
cuckoo -d
```

### Bước 2: Chạy Web Interface

```bash
cuckoo web runserver 0.0.0.0:8000
```

### Bước 3: Truy cập giao diện

```
http://<IP-Host>:8000
```

---

## ✅ Kết quả

* Báo cáo HTML chi tiết
* Log hành vi malware
* Network traffic (pcap)
* Lưu trữ trong MongoDB

---

## 📌 Ghi chú

* Windows 7 được dùng để tăng khả năng kích hoạt hành vi malware
* Luôn dùng **Host-only network** để đảm bảo an toàn
* Snapshot giúp reset môi trường sau mỗi lần phân tích


