# CÁC VÍ DỤ & LỆNH "GỐI ĐẦU GIƯỜNG" CHO DEVOPS

Dưới đây là những tình huống thực tế mà bạn nên note lại vào Repo để sau này troubleshooting.

## Case 1: Kiểm tra "Sức khỏe" Card mạng
- **Lệnh:** `netstat -i`
- **Ví dụ:** Nếu thấy `Ierrs` hoặc `Oerrs` > 0, dây cáp hoặc card mạng của bạn đang bị lỗi vật lý.

## Case 2: Xác định Interface "gánh" Traffic
- **Lệnh:** `netstat -i` (so sánh cột `Ipkts`)
- **Ví dụ:** Như máy bạn, `en7` có triệu gói tin trong khi `en0` chỉ có vài nghìn ➔ `en7` là card mạng chính.

## Case 4: Kiểm tra Port (Cửa vào)
- **Lệnh:** `lsof -nP -iTCP:8080` hoặc `netstat -an | grep 8080`
- **Ví dụ:** Để biết ứng dụng Backend của bạn đã thực sự "chiếm" cổng 8080 chưa.
