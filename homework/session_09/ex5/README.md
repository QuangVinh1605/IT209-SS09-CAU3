# Báo Cáo Thực Hành: Khởi chạy dịch vụ đơn giản với Docker Compose

## Thông tin bài tập
- **Học viên:** Nguyễn Quang Vinh (Vinh Nguyen)
- **Mã lớp:** CNTT1 (DevOps IT209)
- **Môn học:** DevOps (IT209)
- **Bài tập:** Bài 5 (EX5) - Khởi chạy dịch vụ đơn giản với Docker Compose
- **Đường dẫn nộp bài trên GitHub:** `homework/session_09/ex5/`
- **Kho lưu trữ GitHub:** [QuangVinh1605/IT209-SS09-CAU5](https://github.com/QuangVinh1605/IT209-SS09-CAU5)

---

## 1. Mục tiêu bài thực hành
1. **Làm quen với Docker Compose CLI v2:** Chuyển đổi từ phương thức quản lý container thủ công (lệnh `docker run` dài dòng, dễ nhầm lẫn) sang phương pháp **Cơ sở hạ tầng dưới dạng mã (Infrastructure as Code - IaC)** với Docker Compose.
2. **Khai báo tệp `docker-compose.yml` chuẩn cú pháp:** Nắm vững quy tắc định dạng YAML (thụt lề bằng 2 khoảng trắng, không dùng tab).
3. **Quản lý vòng đời dịch vụ bằng Compose CLI:** Sử dụng thành thạo các câu lệnh điều khiển dịch vụ theo nhóm: `up -d`, `ps`, `down`.
4. **Kiểm tra và xác thực kết nối:** Đảm bảo service web khởi chạy chính xác trên cổng `8082` và phản hồi HTTP 200 trang chào mừng Nginx.

---

## 2. Kiến trúc & Sơ đồ luồng dịch vụ (Architecture Flow)

```text
+---------------------------------------------------------------------------------------+
|                                    DOCKER HOST                                        |
|                                                                                       |
|   +--------------------------+  docker compose up -d   +--------------------------+   |
|   |    docker-compose.yml    | ----------------------> | Project Network:         |   |
|   |                          |                         | ex5_default (bridge)     |   |
|   | services:                |                         +--------------------------+   |
|   |   web:                   |                                      |                 |
|   |     image: nginx:alpine  |                                      v                 |
|   |     ports: ["8082:80"]   |                         +--------------------------+   |
|   +--------------------------+                         | Container: ex5-web-1     |   |
|                                                        | Service: web             |   |
|     Host Port: 8082                                    | Image: nginx:alpine      |   |
|     (0.0.0.0:8082)    ======[ Port Mapping ]=========> | Port: 80 (HTTP)          |   |
|            ^                                           +--------------------------+   |
|            |                                                                          |
+------------|--------------------------------------------------------------------------+
             |
       GET / (HTTP)
             |
    +-----------------+
    |  Client / curl  |
    | (localhost:8082)|
    +-----------------+
```

---

## 3. Nội dung tệp cấu hình `docker-compose.yml`

Tệp cấu hình tối giản theo chuẩn Docker Compose Specification:

```yaml
services:
  web:
    image: nginx:alpine
    ports:
      - "8082:80"
```

### Phân tích chi tiết từng khối chỉ thị:
- **`services`**: Khối định nghĩa danh sách các dịch vụ/container cấu thành nên ứng dụng.
- **`web`**: Tên dịch vụ do người dùng đặt (service name). Docker Compose sẽ sử dụng tên này để sinh tên container (`ex5-web-1`) và làm hostname nội bộ phục vụ cho việc phân giải DNS giữa các container.
- **`image: nginx:alpine`**: Chỉ định image nguồn tải từ Docker Hub. Bản Alpine Linux có dung lượng siêu nhẹ (~26MB), giúp tiết kiệm tài nguyên và khởi động gần như tức thì.
- **`ports: - "8082:80"`**: Khai báo quy tắc ánh xạ cổng mạng theo cú pháp `"<CỔNG_HOST>:<CỔNG_CONTAINER>"`. Trong đó, cổng `8082` trên máy chủ host sẽ forward trực tiếp vào cổng `80` của Nginx bên trong container.

---

## 4. Quy trình thực hiện & Nhật ký kiểm tra (Verification Logs)

### Bước 1: Khởi chạy dịch vụ chạy nền (`docker compose up -d`)
```bash
docker compose up -d
```

*Nhật ký thực thi thực tế:*
```text
[+] up 2/2
 ✔ Network ex5_default Created                                              0.6s
 ✔ Container ex5-web-1 Started                                              2.0s
```
> **Đánh giá:** Docker Compose tự động tạo một mạng bridge riêng biệt (`ex5_default`) và khởi chạy container `ex5-web-1` ở chế độ detached (`-d`).

---

### Bước 2: Kiểm tra trạng thái dịch vụ (`docker compose ps`)
```bash
docker compose ps
```

*Nhật ký thực thi thực tế:*
```text
NAME        IMAGE          COMMAND                  SERVICE   CREATED          STATUS          PORTS
ex5-web-1   nginx:alpine   "/docker-entrypoint.…"   web       14 seconds ago   Up 13 seconds   0.0.0.0:8082->80/tcp, [::]:8082->80/tcp
```
> **Đánh giá:** Trạng thái container là `Up`, cổng `8082` đã được liên kết chính xác (`0.0.0.0:8082->80/tcp, [::]:8082->80/tcp`), đúng yêu cầu đề bài.

---

### Bước 3: Kiểm tra phản hồi HTTP từ Nginx (`curl http://localhost:8082`)
```bash
curl http://localhost:8082
```

*Nội dung HTML trả về thực tế:*
```html
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, nginx is successfully installed and working.
Further configuration is required for the web server, reverse proxy, 
API gateway, load balancer, content cache, or other features.</p>

<p>For online documentation and support please refer to
<a href="https://nginx.org/">nginx.org</a>.<br/>
To engage with the community please visit
<a href="https://community.nginx.org/">community.nginx.org</a>.<br/>
For enterprise grade support, professional services, additional 
security features and capabilities please refer to
<a href="https://f5.com/nginx">f5.com/nginx</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

*Kiểm tra thêm HTTP Header (`curl -i http://localhost:8082`):*
```text
HTTP/1.1 200 OK
Server: nginx/1.31.6
Date: Thu, 08 Oct 2026 15:45:06 GMT
Content-Type: text/html
Content-Length: 896
Connection: keep-alive
ETag: "6aa953cc-380"
```
> **Đánh giá:** Nginx phản hồi mã trạng thái `HTTP 200 OK` kèm đầy đủ nội dung trang chào mừng mặc định ("Welcome to nginx!").

---

### Bước 4: Hạ dịch vụ và dọn dẹp môi trường (`docker compose down`)
```bash
docker compose down
```

*Nhật ký thực thi thực tế:*
```text
[+] down 2/2
 ✔ Container ex5-web-1 Removed                                              0.5s
 ✔ Network ex5_default Removed                                              0.3s
```

*Xác nhận lại trạng thái sau khi hạ dịch vụ (`docker compose ps`):*
```bash
docker compose ps
```
*Kết quả:*
```text
NAME      IMAGE     COMMAND   SERVICE   CREATED   STATUS    PORTS
```
> **Đánh giá:** Dịch vụ, container và network tương ứng đã được dừng và thu hồi hoàn toàn khỏi hệ thống một cách an toàn.

---

## 5. Phân tích chuyên sâu (Deep Dive)

### 5.1. So sánh `docker run` thủ công và Docker Compose
| Tiêu chí | `docker run` | Docker Compose |
| :--- | :--- | :--- |
| **Cách tiếp cận** | Mệnh lệnh (Imperative): Gõ chuỗi cờ lệnh phức tạp | Khai báo (Declarative): Mô tả trạng thái mong muốn trong file YAML |
| **Khả năng tái lập** | Khó nhớ, dễ sai sót tham số giữa các môi trường | Dễ lưu trữ vào Git, đảm bảo môi trường giống nhau 100% |
| **Mở rộng nhiều dịch vụ** | Phải gõ từng lệnh, tự tạo network thủ công | Quản lý đồng thời hàng chục service chỉ bằng 1 lệnh `up` / `down` |
| **Quản lý mạng & DNS** | Cần `docker network create` và gán tay | Tự động tạo network và cấu hình Service Discovery qua DNS nội bộ |

### 5.2. Tại sao tệp `docker-compose.yml` không cần khai báo `version`?
- Trong chuẩn **Compose Specification** hiện đại (Docker Compose CLI v2 - lệnh `docker compose`), thuộc tính cấp cao `version: '3.8'` hoặc `version: '3'` đã được xem là **không bắt buộc và đã lỗi thời (deprecated)**.
- Compose Engine tự động áp dụng phiên bản schema mới nhất phù hợp với Docker Engine trên máy chủ, giúp cấu hình ngắn gọn và tương thích tốt hơn.

### 5.3. Quy tắc định dạng YAML quan trọng
- Sử dụng đúng **2 dấu khoảng trắng** cho mỗi cấp thụt lề (indentation).
- **Tuyệt đối không dùng phím Tab** vì bộ phân tích cú pháp YAML sẽ báo lỗi `yaml: found character that cannot start any token`.
- Các giá trị cổng dạng số chứa dấu `:` nên được đặt trong dấu ngoặc kép (ví dụ: `"8082:80"`) để tránh bị YAML parser hiểu nhầm sang kiểu dữ liệu số hoặc thời gian (sexagesimal).

---

## 6. Danh mục tệp nộp bài

- [docker-compose.yml](file:///home/vinh/Desktop/devoops/ss08/homework/session_09/ex5/docker-compose.yml): Tệp cấu hình Docker Compose khai báo dịch vụ web.
- [README.md](file:///home/vinh/Desktop/devoops/ss08/homework/session_09/ex5/README.md): Báo cáo chi tiết quá trình khởi chạy, kiểm tra và dọn dẹp dịch vụ.
