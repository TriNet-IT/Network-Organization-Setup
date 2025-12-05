# Triển khai Mạng Doanh nghiệp Lớp 2/Lớp 3 

## Tổng quan Dự án (Project Overview)

Dự án này là bài kiểm tra toàn diện cuối khóa **CCNA: Routing and Switching Essentials**, thể hiện khả năng **thiết kế, cấu hình và bảo trì** một mạng doanh nghiệp đa tầng.

### Mục tiêu Chính

Mục tiêu trọng tâm của dự án là triển khai một kiến trúc mạng đáp ứng các tiêu chí sau:

* **Dự phòng (Redundancy):** Đảm bảo tính sẵn sàng cao thông qua các giao thức dự phòng Gateway (như HSRP).
* **Bảo mật (Security):** Áp dụng các chính sách an ninh mạng phức tạp (như ACL, Port Security) để kiểm soát truy cập và bảo vệ mạng.
* **Hiệu suất (Efficiency):** Tối ưu hóa việc sử dụng địa chỉ IP bằng **VLSM** và sử dụng định tuyến động để đảm bảo hội tụ nhanh chóng.
* Dự án bao gồm xử lý các tác vụ từ việc chia subnet bằng VLSM đến áp dụng các chính sách an ninh mạng phức tạp, xác nhận khả năng sẵn sàng cho vai trò kỹ sư mạng.
---

## Các Tính năng Kỹ thuật Đã Triển khai (Core Implementation)

Dự án này bao gồm **10 phần cấu hình cốt lõi**, tập trung vào việc áp dụng các tiêu chuẩn công nghiệp:

| Lĩnh vực | Công nghệ | Chi tiết Triển khai |
| :--- | :--- | :--- |
| **Phân bổ IP** | **VLSM** | Chia lớp mạng được cấp phát thành các subnet tối ưu (VLAN 10, 20, 60, 61, 90). |
| **Dự phòng** | **HSRP** (Hot Standby Router Protocol) | Triển khai Gateway dự phòng trên Switch L3 (**S11** và **S12**) để đảm bảo tính sẵn sàng cao (High Availability). |
| **Chuyển mạch L2** | **VLAN, Trunking, EtherChannel** | Tách biệt các miền broadcast, cấu hình **Trunking 802.1Q** và gộp liên kết giữa các Switch L3 bằng **LACP**. |
| **Bảo mật L2** | **Port Security & Portfast** | Cấu hình **Port Security** (Giới hạn 2 địa chỉ MAC) với hành động vi phạm **Shutdown** trên các cổng Access; Tối ưu hóa STP bằng Portfast. |
| **Định tuyến L3** | **OSPF (Single Area)** | Định tuyến động sử dụng OSPF để quảng bá tất cả các mạng nội bộ và học/quảng bá default route ra Router Internet. |
| **Dịch vụ Mạng** | **DHCP Relay** | Cấu hình DHCP Server trên máy chủ và **DHCP Relay** trên Switch L3 để cung cấp IP động cho VLAN 10 và VLAN 20. |
| **Dịch Thuật** | **NAT (Dynamic & Static)** | Cấu hình **Dynamic NAT** cho toàn bộ mạng LAN ra Internet và **Static NAT** để public dịch vụ Web Server vào mạng ngoài. |
| **Kiểm soát Truy cập** | **Extended ACL** | **Chính sách ACL nghiêm ngặt** để: (1) Chỉ VLAN10 SSH đến thiết bị mạng, (2) Ngăn chặn VLAN10/20 giao tiếp với nhau, (3) Chỉ cho phép Server HTTP/HTTPS ra Internet. |

---

## Mô hình và Cấu trúc Mạng



<img width="934" height="666" alt="image" src="https://github.com/user-attachments/assets/364dc533-1a10-4d59-86bb-95e4c3a6b335" />




### Thiết bị Chính (Core Devices)

* **Router (R1):** Điểm kết nối Internet, thực hiện NAT và Định tuyến OSPF.
* **Distribution/Core (S11, S12):** Switch L3, thực hiện Inter-VLAN Routing, HSRP, và DHCP Relay.
* **Access (S21, S22, S23, S24):** Switch L2, thực hiện VLAN Assignment, Port Security.
* **Servers:** DHCP Server, Web Server.

---

## Thông tin Kỹ thuật Quan trọng

### 1. Địa chỉ IP và Default Gateway (HSRP)

Dự án được xây dựng trên lớp mạng cơ sở được chỉ định ở phụ lục.

| Subnet | Địa chỉ Mạng/CIDR | Default Gateway (HSRP Virtual IP) | Vai trò Active |
| :--- | :--- | :--- | :--- |
| **VLAN 10** | `10.16.16.64/26` | `10.16.16.65` | S11 |
| **VLAN 20** | `10.16.16.0/26` | `10.16.16.1` | S11 |
| **VLAN 60** | `10.16.16.128/28` | `10.16.16.129` | S12 |
| **VLAN 61** | `10.16.16.144/28` | `10.16.16.145` | S12 |

### 2. Ví dụ Chính sách ACL

Để kiểm soát lưu lượng nghiêm ngặt, các ACL sau đã được áp dụng:

#### Bảo mật SSH (Extended ACL 10):

Chỉ mạng **VLAN10** (`10.16.16.64/26`) được phép SSH (Port 22) tới các thiết bị mạng.

```text
access-list 10 permit tcp 10.16.16.64 0.0.0.63 any eq 22
```

#### Ngăn chặn Inter-VLAN (Extended ACL 20):

Từ chối lưu lượng giữa VLAN10 và VLAN20, được gắn theo chiều OUT trên Interface VLAN tương ứng.

```text
access-list 20 deny ip 10.16.16.64 0.0.0.63 10.16.16.0 0.0.0.63
access-list 20 deny ip 10.16.16.0 0.0.0.63 10.16.16.64 0.0.0.63
```

---

## Liên hệ:
* Tác giả: Nguyễn Phương Minh Trí
* Email: ngphtri15@gmail.com

