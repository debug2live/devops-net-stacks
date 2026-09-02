# KẾ HOẠCH LUYỆN THI CKA - 47 NGÀY

---

## 🟥 GIAI ĐOẠN 1: STORAGE & WORKLOADS (NGÀY 1 - 12)
**Mục tiêu:** 30% điểm  
**Mantra:** Không tự viết YAML. Luôn luôn dùng lệnh imperative + `$do` để sinh cấu trúc.

---

### 📦 Tuần 1: Trận Chiến Lưu Trữ (Storage Deep Dive)

- **Ngày 1:** Master lệnh tạo StorageClass (SC) thủ công & động. Phân biệt ReclaimPolicy (Delete vs Retain).

- **Ngày 2:** Tạo PersistentVolume (PV) dạng hostPath. Ép bản thân nhớ các thuộc tính: accessModes, capacity, volumeMode.

- **Ngày 3:** Tạo PersistentVolumeClaim (PVC) và thực hành "gán ghép" (Bound) PVC vào PV. Xử lý lỗi nếu PVC bị treo ở trạng thái Pending.

- **Ngày 4:** Tạo Pod/Deployment, cấu hình volumes và volumeMounts để nhúng PVC vào container. Kiểm tra ghi dữ liệu ra Node.

---

### 🎮 Tuần 2: Điều Phối Quân Cờ (Scheduling & Workloads)

- **Ngày 5:** Làm chủ nodeSelector và nodeName để chỉ định chính xác Pod phải chạy trên Worker Node nào.

- **Ngày 6:** Nâng cấp lên NodeAffinity (Cấu hình requiredDuringScheduling... và preferredDuringScheduling...).

- **Ngày 7:** Đóng vai "Kẻ gác cổng": Cấu hình Taints trên Node (Dùng NoSchedule, NoExecute) và viết Tolerations cho Pod để vượt rào.

- **Ngày 8:** Biến đổi YAML: Luyện kỹ năng chuyển một file Deployment thành DaemonSet (Xóa replicas, thay đổi Kind) để chạy trên mọi Node.

- **Ngày 9:** Thiết kế Multi-Container Pod: Cấu hình initContainers để chuẩn bị dữ liệu trước khi container chính khởi chạy.

- **Ngày 10:** Cấu hình Sidecar Container: Chạy một container phụ (như busybox) để thu thập và streaming log từ container chính.

- **Ngày 11:** Tổng duyệt Giai đoạn 1: Tự đặt đề bài ngẫu nhiên, phối hợp PV + Taints + Sidecar trên cùng một Pod.

- **Ngày 12:** 🔥 **NGÀY CHECK-POINT 1:** Tự kiểm tra tốc độ. Phải sinh file YAML cho Deployment + Service trong 45 giây. **[ĐẠT ➔ QUA GĐ 2]**

---

## 🟨 GIAI ĐOẠN 2: NETWORKING & SECURITY (NGÀY 13 - 24)
**Mục tiêu:** 30% điểm  
**Mantra:** Một dấu gạch đầu dòng (-) sai vị trí trong NetworkPolicy sẽ cô lập toàn bộ Cluster.

---

### 🌐 Tuần 3: Thiết Lập Ma Trận Mạng (Services & Ingress)

- **Ngày 13:** Phân biệt và cấu hình Service ClusterIP và NodePort. Thuộc lòng cách mapping port vs targetPort.

- **Ngày 14:** Cài đặt Ingress Controller trên Multipass. Viết Ingress Resource để định tuyến traffic dựa trên /path (Routing).

- **Ngày 15:** Nâng cao Ingress: Cấu hình Ingress Host-based (domain.com) và chuẩn bị sẵn tư duy cho bài thi cấu hình Ingress TLS.

- **Ngày 16:** Trọng tâm CKA: Viết NetworkPolicy cô lập mạng. Chặn toàn bộ traffic đi vào namespace (Default Deny All).

- **Ngày 17:** Viết NetworkPolicy nâng cao: Chỉ cho phép Pod có nhãn app=frontend gọi Pod app=backend qua một Port nhất định.

- **Ngày 18:** Thực hành cấu hình Egress (Traffic đi ra ngoài Cluster) kết hợp với ipBlock.

---

### 🛡️ Tuần 4: Thiết Lập Thiết Quân Luật (Security & RBAC)

- **Ngày 19:** Làm quen với ServiceAccount (SA). Hiểu cách Pod sử dụng SA để giao tiếp với Kube-APIServer.

- **Ngày 20:** Tạo Role (Giới hạn trong Namespace) và ClusterRole (Toàn cục Cluster). Định nghĩa chính xác resources và verbs.

- **Ngày 21:** Thắt chặt vòng vây bằng RoleBinding và ClusterRoleBinding. Gán quyền của ClusterRole cho một SA cụ thể.

- **Ngày 22:** Luyện lệnh kiểm tra quyền: `kubectl auth can-i <action> <resource> --as=system:serviceaccount:...`

- **Ngày 23:** Cấu hình SecurityContext mức Pod và Mức Container. Ép container chạy với runAsUser: 2000 (Non-root).

- **Ngày 24:** 🔥 **NGÀY CHECK-POINT 2:** Giả lập chạy lệnh kiểm tra NetworkPolicy độc lập từ Pod test. Nếu chặn thành công ➔ **[PASS ➔ QUA GĐ 3]**

---

## 🟧 GIAI ĐOẠN 3: ARCHITECTURE & TROUBLESHOOTING (NGÀY 25 - 37)
**Mục tiêu:** 40% điểm  
**Mantra:** Khi Control Plane sập, lệnh kubectl vô dụng. Hãy tin vào systemctl và crictl.

---

### 🛠️ Tuần 5: Bảo Trì & Nâng Cấp Đội Hình (Cluster Maintenance)

- **Ngày 25:** Thực hành lệnh `kubectl cordon` (Khóa node) và `kubectl drain` (Đuổi Pod an toàn để bảo trì Node).

- **Ngày 26:** Cực kỳ quan trọng: Backup dữ liệu ETCD. Thuộc lòng cú pháp sử dụng `--cert`, `--key`, `--cacert` của lệnh etcdctl.

- **Ngày 27:** Thực hành Restore dữ liệu ETCD từ file snapshot vừa backup. Khởi động lại kubelet để nhận cụm mới.

- **Ngày 28:** Học quy trình nâng cấp Cluster: Nâng cấp công cụ kubeadm trên Master Node đầu tiên.

- **Ngày 29:** Chạy lệnh `kubeadm upgrade plan` và `kubeadm upgrade apply`. Sau đó nâng cấp kubelet và kubectl.

- **Ngày 30:** Lặp lại quy trình nâng cấp cho các Worker Nodes (`kubeadm upgrade node`).

---

### 🚨 Tuần 6: Khắc Phục Sự Cố Chí Mạng (Troubleshooting)

- **Ngày 31:** Giả lập Node bị NotReady. SSH vào node, dùng `systemctl status kubelet` và `journalctl -u kubelet -f` để tìm lỗi driver/config.

- **Ngày 32:** Điều tra lỗi Container Runtime sập. Sử dụng `crictl ps -a` và `crictl logs` trực tiếp dưới OS để cứu containerd.

- **Ngày 33:** Tấn công vào Control Plane: Phá hoại file manifest tĩnh `/etc/kubernetes/manifests/kube-apiserver.yaml`. Sửa lỗi để hồi sinh API Server.

- **Ngày 34:** Debug lỗi ứng dụng: Sửa các lỗi CrashLoopBackOff, ImagePullBackOff bằng cách đọc `kubectl describe` và `kubectl logs`.

- **Ngày 35:** Khắc phục sự cố mạng và phân giải tên miền: Kiểm tra log của Pod coredns trong namespace kube-system.

- **Ngày 36:** Luyện tập tìm kiếm lỗi sai thông tin cấu hình trong các file kubeconfig (`/etc/kubernetes/admin.conf`).

- **Ngày 37:** 🔥 **NGÀY CHECK-POINT 3:** Tự "bẻ khóa" một cụm cluster bị sửa sai certificate trong vòng 10 phút không cần trợ giúp. **[PASS ➔ QUA GĐ 4]**

---

## 🟦 GIAI ĐOẠN 4: KILLER.SH & FINAL SPRINT (NGÀY 38 - 47)
**Mục tiêu:** Tốc độ và Chiến thắng  
**Mantra:** Vào phòng thi: Việc đầu tiên là cài alias, việc thứ hai là check context!

---

- **Ngày 38:** KÍCH HOẠT KILLER.SH LƯỢT 1. Chấp nhận bị ngợp trước 25 câu hỏi siêu khó. Làm bài nghiêm túc trong 120 phút.

- **Ngày 39:** Đóng cửa tự suy ngẫm: Đọc toàn bộ đáp án của Killer.sh. Ghi lại các câu bị 0 điểm vào Obsidian.

- **Ngày 40:** Cày lại các bài lab của những câu làm sai. Thực hành đi thực hành lại cho thuộc giải pháp.

- **Ngày 41:** Luyện kỹ năng "Săn tài liệu" (Documentation Hunting). Đặt giờ tìm một đoạn code mẫu trên trang chủ K8s trong dưới 15 giây.

- **Ngày 42:** KÍCH HOẠT KILLER.SH LƯỢT 2. Làm lại đề cũ với mục tiêu tối thượng: Tốc độ. Đạt mục tiêu > 85 điểm.

- **Ngày 43:** Luyện thuộc lòng bộ khung gõ tắt Aliases (k, do, now) để biến nó thành phản xạ vô điều kiện.

- **Ngày 44:** Dọn dẹp chiến trường: Kiểm tra hộ chiếu (Passport), dọn sạch bàn học, test camera/mic theo link của PSI Test Center.

- **Ngày 45:** Tổng rà soát lại ma trận điểm, thư giãn đầu óc, đi ngủ sớm.