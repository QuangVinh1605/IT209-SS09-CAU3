# Báo Cáo Thực Hành: Đóng gói trang Web HTML tĩnh với Dockerfile Nginx Alpine

## Thông tin bài tập
- **Học viên:** Nguyễn Quang Vinh (Vinh Nguyen)
- **Mã lớp:** CNTT1 (DevOps IT209)
- **Môn học:** DevOps (IT209)
- **Bài tập:** Bài 3 (EX3) - Viết Dockerfile đóng gói trang Web HTML tĩnh
- **Đường dẫn nộp bài trên GitHub:** `homework/session_09/ex3/`
- **Kho lưu trữ GitHub:** [QuangVinh1605/IT209-SS09-CAU3](https://github.com/QuangVinh1605/IT209-SS09-CAU3)

---

## 1. Mục tiêu bài thực hành
1. **Làm quen với Dockerfile:** Hiểu cách khai báo cấu hình tự động đóng gói ứng dụng web vào Docker Image.
2. **Sử dụng Base Image tối ưu (`nginx:alpine`):** Tận dụng hệ điều hành Alpine Linux siêu nhẹ để tạo ra Image có dung lượng tối giản (~26MB), khởi động gần như tức thì và giảm thiểu bề mặt tấn công bảo mật.
3. **Cơ chế nạp file (`COPY`):** Đưa trang tĩnh `index.html` vào đúng thư mục Document Root mặc định của Nginx (`/usr/share/nginx/html/index.html`).
4. **Quản lý Vòng đời Container & Port Mapping:** Thực hiện build image, chạy container ngầm (`-d`) và chuyển tiếp cổng mạng (`-p 8081:80`) để client có thể truy cập từ host.

---

## 2. Kiến trúc & Sơ đồ luồng hoạt động (Architecture & Flow)

```text
+---------------------------------------------------------------------------------------+
|                                    DOCKER HOST                                        |
|                                                                                       |
|   +-------------------+    docker build     +-------------------------------------+   |
|   |    Dockerfile     | ------------------> |          my-html-app:v1             |   |
|   |    index.html     |                     | (Base: nginx:alpine - Size ~26.3MB) |   |
|   +-------------------+                     +-------------------------------------+   |
|                                                                |                      |
|                                                           docker run                  |
|                                                                v                      |
|                                                     +--------------------+            |
|     Host Port: 8081                                 | Container: html-app|            |
|     (0.0.0.0:8081)    ==[ Port Forwarding -p ]===>  | (Nginx Port 80)    |            |
|            ^                                        | Document Root:     |            |
|            |                                        | /usr/share/nginx/  |            |
|            |                                        | html/index.html    |            |
|            |                                        +--------------------+            |
|            |                                                                          |
+------------|--------------------------------------------------------------------------+
             |
       GET / (HTTP)
             |
    +-----------------+
    |  Client / curl  |
    | (localhost:8081)|
    +-----------------+
```

---

## 3. Nội dung các tệp mã nguồn

### 3.1. Tệp `index.html`
Nội dung tệp tĩnh hiển thị thông báo theo yêu cầu:
```html
<h1>Hello Docker Session 09!</h1>
```

### 3.2. Tệp `Dockerfile`
Tệp cấu hình đóng gói ngắn gọn gồm đúng 2 câu lệnh:
```dockerfile
FROM nginx:alpine
COPY index.html /usr/share/nginx/html/index.html
```

#### Phân tích chi tiết các chỉ thị:
- **`FROM nginx:alpine`**: Sử dụng base image Nginx chạy trên nền tảng Alpine Linux. Image này đã được cài đặt sẵn máy chủ web Nginx và cấu hình mặc định tối ưu, dung lượng sau tải về chỉ khoảng ~26MB.
- **`COPY index.html /usr/share/nginx/html/index.html`**: Sao chép tệp `index.html` từ thư mục làm việc hiện tại trên máy host (build context) vào thư mục Document Root mặc định của Nginx Alpine (`/usr/share/nginx/html/`), thay thế cho tệp `index.html` mặc định ("Welcome to nginx!").

---

## 4. Các bước triển khai và kiểm tra

### Bước 1: Build Docker Image
Chạy lệnh build image với tên `my-html-app` và tag `v1`:
```bash
docker build -t my-html-app:v1 .
```

*Kết quả build:*
```text
[+] Building 0.6s (7/7) FINISHED                                 docker:default
 => [internal] load build definition from Dockerfile                       0.0s
 => => transferring dockerfile: 104B                                       0.0s
 => [internal] load metadata for docker.io/library/nginx:alpine            0.0s
 => [internal] load .dockerignore                                          0.0s
 => => transferring context: 2B                                            0.0s
 => [internal] load build context                                          0.0s
 => => transferring context: 71B                                           0.0s
 => CACHED [1/2] FROM docker.io/library/nginx:alpine                       0.0s
 => [2/2] COPY index.html /usr/share/nginx/html/index.html                 0.1s
 => exporting to image                                                     0.2s
 => => exporting layers                                                    0.1s
 => => naming to docker.io/library/my-html-app:v1                          0.0s
 => => unpacking to docker.io/library/my-html-app:v1                       0.0s
```

### Bước 2: Khởi chạy Container
Khởi chạy container chạy nền (`-d`), ánh xạ cổng host 8081 vào cổng 80 của container (`-p 8081:80`), đặt tên container là `html-app`:
```bash
docker run -d -p 8081:80 --name html-app my-html-app:v1
```

*Container ID sinh ra:*
```text
1cc987a96f1e70d639d1b613d81bc60ce0399d468a006ff88c925b549af2acc5
```

### Bước 3: Kiểm tra trạng thái Container đang chạy
Kiểm tra danh sách container hoạt động:
```bash
docker ps --filter "name=html-app"
```

*Kết quả:*
```text
CONTAINER ID   IMAGE            COMMAND                  CREATED          STATUS          PORTS                                     NAMES
1cc987a96f1e   my-html-app:v1   "/docker-entrypoint.…"   12 seconds ago   Up 11 seconds   0.0.0.0:8081->80/tcp, [::]:8081->80/tcp   html-app
```

---

## 5. Nhật ký kiểm tra & Xác thực kết quả (Verification Logs)

### 5.1. Kiểm tra HTTP Response Body (`curl http://localhost:8081`)
```bash
curl http://localhost:8081
```

*Kết quả đầu ra:*
```html
<h1>Hello Docker Session 09!</h1>
```
> **Đánh giá:** Lệnh curl trả về chính xác chuỗi `<h1>Hello Docker Session 09!</h1>` theo đúng kết quả mong đợi của đề bài.

---

### 5.2. Kiểm tra HTTP Response Header (`curl -i http://localhost:8081`)
```bash
curl -i http://localhost:8081
```

*Kết quả đầu ra:*
```text
HTTP/1.1 200 OK
Server: nginx/1.31.6
Date: Thu, 08 Oct 2026 15:05:28 GMT
Content-Type: text/html
Content-Length: 34
Last-Modified: Thu, 08 Oct 2026 15:05:05 GMT
Connection: keep-alive
ETag: "6ac7b121-22"
Accept-Ranges: bytes

<h1>Hello Docker Session 09!</h1>
```
> **Đánh giá:** Server phản hồi mã trạng thái `HTTP 200 OK`, định dạng `text/html`, server là `nginx/1.31.6`.

---

### 5.3. Kiểm tra Logs của Container (`docker logs html-app`)
```bash
docker logs html-app
```

*Trích xuất nhật ký truy cập (Access Log):*
```text
172.17.0.1 - - [08/Oct/2026:15:05:25 +0000] "GET / HTTP/1.1" 200 34 "-" "curl/8.18.0" "-"
172.17.0.1 - - [08/Oct/2026:15:05:28 +0000] "GET / HTTP/1.1" 200 34 "-" "curl/8.18.0" "-"
```
> **Đánh giá:** Nginx trong container đã tiếp nhận thành công các HTTP GET request từ máy host và phản hồi mã `200`.

---

## 6. Phân tích chuyên sâu (Deep Dive)

### 6.1. Tại sao nên sử dụng Base Image `nginx:alpine`?
- **Kích thước siêu nhẹ (Lightweight):** Image `nginx:alpine` chỉ chiếm khoảng **~26.3 MB**, trong khi bản `nginx:latest` (dựa trên Debian) có kích thước lên tới **~190 MB**.
- **Tốc độ triển khai (Fast Deployment):** Thời gian tải (pull) và khởi động container gần như tức thì, tiết kiệm băng thông mạng và dung lượng lưu trữ trong các pipeline CI/CD.
- **Bảo mật cao hơn (Reduced Attack Surface):** Alpine Linux sử dụng thư viện C tối giản `musl libc` và `BusyBox`, loại bỏ hầu hết các package/công cụ không cần thiết như `curl`, `bash`, `perl`, từ đó giảm thiểu đáng kể số lượng lỗ hổng CVE tiềm ẩn.

### 6.2. Cơ chế Port Forwarding `-p 8081:80`
- Bên trong container, Nginx mặc định lắng nghe tại cổng HTTP chuẩn `80`.
- Tuy nhiên, mạng của container là mạng nội bộ (bridge network). Để bên ngoài (máy host hoặc máy khác trong mạng LAN) có thể truy cập được, cờ `-p 8081:80` sẽ thiết lập quy tắc iptables / NAT:
  - Mọi gói tin TCP gửi đến `0.0.0.0:8081` trên máy host sẽ được tự động chuyển hướng (DNAT) đến địa chỉ IP nội bộ của container tại cổng `80`.

---

## 7. Hướng dẫn dọn dẹp tài nguyên (Cleanup Commands)
Khi hoàn thành kiểm tra và muốn giải phóng tài nguyên hệ thống:
```bash
# Dừng container
docker stop html-app

# Xóa container
docker rm html-app

# Xóa image vừa tạo (nếu không còn sử dụng)
docker rmi my-html-app:v1
```

---

## 8. Danh mục tệp nộp bài

- [index.html](file:///home/vinh/Desktop/devoops/ss08/homework/session_09/ex3/index.html): Trang HTML tĩnh hiển thị thông điệp.
- [Dockerfile](file:///home/vinh/Desktop/devoops/ss08/homework/session_09/ex3/Dockerfile): Tệp cấu hình đóng gói image Nginx Alpine.
- [README.md](file:///home/vinh/Desktop/devoops/ss08/homework/session_09/ex3/README.md): Báo cáo chi tiết quá trình đóng gói, chạy thử nghiệm và kiểm tra.
