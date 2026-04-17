# DevOps Networking Roadmap

Chào mừng bạn đến với kho lưu trữ (repository) dành cho việc học Networking từ góc nhìn của một Backend Developer hướng tới DevOps. Repo này cung cấp lộ trình học tập từ cơ bản đến nâng cao, đi kèm các bài tập thực hành (Labs) để củng cố lý thuyết.

## 🚀 Lộ Trình Học Tập (Roadmap Checklist)

### LEVEL 1: NỀN TẢNG "DÂY DẪN" (CORE FOUNDATIONS)

**IP Addressing & Routing**
- [ ] Hiểu IPv4, Subnet Mask và CIDR (Ví dụ: `/24`, `/16` là gì).
- [ ] Phân biệt Public IP vs Private IP (RFC1918).
- [ ] Cách Router quyết định đường đi (Routing Table).

**Protocols (Giao thức)**
- [ ] TCP vs UDP (Three-way handshake, Window size).
- [ ] DNS (Cách một domain biến thành IP).
- [ ] ICMP (Ping, Traceroute).

**Application Layer**
- [ ] HTTP/1.1 vs HTTP/2 vs HTTP/3.
- [ ] TLS/SSL Handshake (Cách chứng chỉ HTTPS hoạt động).

---

### LEVEL 2: LINUX NETWORKING (THE DEVOPS PLAYGROUND)

**Interface & Virtualization**
- [ ] Ethernet interfaces (`eth0`, `ens3`).
- [ ] Virtual Ethernet (`veth`) - Cực kỳ quan trọng để hiểu Container.
- [ ] Network Namespaces (Cách Linux cô lập mạng).

**Traffic Control & Security**
- [ ] Iptables / NFTables (Lọc gói tin).
- [ ] IP Forwarding (Biến Linux thành router).
- [ ] Port Forwarding & NAT.

---

### LEVEL 3: CONTAINER & ORCHESTRATION (K8S NETWORKING)

**Docker Networking**
- [ ] Bridge Mode (Mặc định).
- [ ] Host Mode.
- [ ] Overlay Network (Mạng giữa nhiều server).

**Kubernetes Core**
- [ ] Pod Networking (Mỗi Pod một IP).
- [ ] Service Types (ClusterIP, NodePort, LoadBalancer).
- [ ] Ingress Controllers (Lớp 7 - Routing dựa trên Path/Domain).
- [ ] CoreDNS trong K8s.

---

### LEVEL 4: CLOUD & ADVANCED (ENTERPRISE GRADE)

**Cloud Provider Networking (AWS/Azure/GCP)**
- [ ] VPC (Virtual Private Cloud).
- [ ] Subnetting (Public vs Private subnets).
- [ ] Security Groups vs Network ACLs.

**Connectivity**
- [ ] Site-to-Site VPN.
- [ ] VPC Peering / Transit Gateway.

**Modern Concepts**
- [ ] Service Mesh (Istio, Linkerd) - Quản lý traffic giữa các microservices.
- [ ] eBPF (Công nghệ networking hiện đại nhất hiện nay).

---

## 🛠 Các Dạng Demo Thực Hành (Labs)

Để dễ hình dung và nắm vững kiến thức, sau mỗi module lý thuyết sẽ có các bài Lab thực hành.

1. **Demo DNS & HTTP**
   - Dùng `dig +trace google.com` để xem gói tin đi qua những máy chủ tên miền nào.
   - Dùng `openssl s_client -connect google.com:443` để xem quá trình trao đổi Certificate.

2. **Demo Linux Namespace (Mô phỏng Container mạng)**
   - Tự tạo 2 namespace trên máy Linux.
   - Tạo cặp dây `veth`, gắn mỗi đầu vào 1 namespace và thử ping thông qua đó. Đây là cách Docker hoạt động dưới "nắp capo".

3. **Demo K8s Service**
   - Deploy 2 ứng dụng khác nhau, dùng Service để chúng gọi nhau bằng tên thay vì dùng IP.
   - Sau đó xóa Pod để thấy Service IP vẫn giữ nguyên nhưng Pod IP thay đổi (Hiểu về tính trừu tượng).

## 📂 Cấu Trúc Thư Mục

- [`01_Core_Foundations/`](01_Core_Foundations/README.md): Tài liệu và Labs cho Level 1.
- [`02_Linux_Networking/`](02_Linux_Networking/README.md): Tài liệu và Labs cho Level 2.
- [`03_Container_Orchestration/`](03_Container_Orchestration/README.md): Tài liệu và Labs cho Level 3.
- [`04_Cloud_Advanced/`](04_Cloud_Advanced/README.md): Tài liệu và Labs cho Level 4.
