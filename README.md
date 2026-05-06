# Tài Liệu Hướng Dẫn - Spring Boot Digital Ocean Test

Tài liệu này hướng dẫn cách build, chạy ứng dụng ở môi trường phát triển (Dev), môi trường thực tế (Production), và cách deploy lên nền tảng Digital Ocean.

## 1. Yêu cầu hệ thống (Prerequisites)
- **Java Development Kit (JDK) 17** hoặc mới hơn.
- Không yêu cầu cài đặt Maven (dự án đã tích hợp sẵn Maven Wrapper `mvnw`).
- Cài đặt Git (để đẩy code và deploy lên Digital Ocean).

---

## 2. Môi trường Phát triển (Dev)

Trong quá trình phát triển, bạn có thể chạy ứng dụng trực tiếp bằng Spring Boot plugin để dễ dàng test code nhanh chóng.

### 2.1. Build dự án
Mở terminal tại thư mục gốc của dự án (`digital-ocean-test`) và chạy lệnh sau để tải các thư viện và compile code:
- **Trên Windows:**
  ```cmd
  .\mvnw.cmd clean compile
  ```
- **Trên Linux/macOS:**
  ```bash
  ./mvnw clean compile
  ```

### 2.2. Chạy dự án (Start)
Để khởi động server ở chế độ dev:
- **Trên Windows:**
  ```cmd
  .\mvnw.cmd spring-boot:run
  ```
- **Trên Linux/macOS:**
  ```bash
  ./mvnw spring-boot:run
  ```

Sau khi chạy thành công, ứng dụng sẽ khởi động ở địa chỉ: `http://localhost:8080`

---

## 3. Môi trường Thực tế (Production)

Ở môi trường Production, ứng dụng nên được build thành một tệp thực thi duy nhất (`.jar`) và chạy độc lập. Tệp `.jar` này đã chứa sẵn web server (Tomcat) bên trong.

### 3.1. Build tệp .jar
Lệnh này sẽ đóng gói toàn bộ ứng dụng và các thư viện liên quan thành một file `.jar` duy nhất:
- **Trên Windows:**
  ```cmd
  .\mvnw.cmd clean package
  ```
- **Trên Linux/macOS:**
  ```bash
  ./mvnw clean package
  ```
Sau khi lệnh chạy xong, bạn sẽ tìm thấy tệp `.jar` trong thư mục `target/` (ví dụ: `target/digital-ocean-test-0.0.1-SNAPSHOT.jar`).

### 3.2. Chạy ứng dụng (Start)
Di chuyển file `.jar` lên server (hoặc chạy tại server production) bằng lệnh Java tiêu chuẩn:
```bash
java -jar target/digital-ocean-test-0.0.1-SNAPSHOT.jar
```
*(Lưu ý: Bạn có thể thay đổi port chạy thực tế bằng cách thêm `--server.port=80` vào cuối lệnh nếu cần).*

---

## 4. Hướng dẫn Deploy lên Digital Ocean App Platform

Digital Ocean App Platform là cách đơn giản nhất để host ứng dụng Spring Boot. Nền tảng này tự động nhận diện dự án Maven và tự động build/deploy mà không cần bạn phải tự viết Dockerfile hay cấu hình server thủ công.

### Bước 4.1: Đẩy code lên Git (GitHub / GitLab)
Digital Ocean App Platform lấy mã nguồn trực tiếp từ kho lưu trữ Git của bạn để deploy.
1. Khởi tạo Git trong thư mục dự án của bạn:
   ```bash
   git init
   git add .
   git commit -m "Initial commit cho Digital Ocean"
   ```
2. Tạo một repository mới trên GitHub (hoặc GitLab) và đẩy code lên:
   ```bash
   git branch -M main
   git remote add origin <URL_CỦA_REPOSITORY_TRÊN_GITHUB>
   git push -u origin main
   ```

### Bước 4.2: Tạo App trên Digital Ocean
1. Đăng nhập vào [Digital Ocean Dashboard](https://cloud.digitalocean.com/).
2. Nhấn nút **Create** (ở góc trên bên phải màn hình) và chọn **Apps**.
3. Tại phần **Choose Source**, chọn **GitHub** (hoặc GitLab tuỳ nơi bạn lưu code).
4. Cấp quyền truy cập cho Digital Ocean và chọn repository bạn vừa tạo ở *Bước 4.1*. Bấm **Next**.

### Bước 4.3: Cấu hình App
1. Digital Ocean sẽ tự động quét mã nguồn và nhận diện đây là một ứng dụng **Java** sử dụng **Maven**.
2. Tại phần **Resources**, bạn có thể chỉnh sửa tên ứng dụng cho đẹp (tuỳ chọn).
3. Tại phần **Environment Variables** (Biến môi trường): Nếu ứng dụng cần kết nối database sau này, bạn sẽ thêm cấu hình ở đây. Hiện tại ứng dụng cơ bản chưa cần nên có thể bỏ qua.
4. **HTTP Port**: Spring Boot mặc định chạy ở port **8080**, Digital Ocean sẽ tự động nhận diện port này. Nếu hệ thống hỏi, hãy chắc chắn HTTP Port là `8080`.
5. Bấm **Next** để tới phần thanh toán (chọn cấu hình máy chủ Starter/Basic phù hợp với nhu cầu).
6. Bấm **Build and Deploy**.

### Bước 4.4: Hoàn tất và Kiểm tra
- Digital Ocean sẽ tự động tạo một máy chủ ẩn, chạy lệnh `mvnw clean package` và khởi động file `.jar` cho bạn.
- Quá trình này mất khoảng 2-5 phút. Khi thanh tiến trình hoàn tất và trạng thái chuyển sang màu xanh **Live**, bạn sẽ được cấp một đường link URL công khai (ví dụ: `https://digital-ocean-test-abcd.ondigitalocean.app`).
- Truy cập vào link đó trên trình duyệt, bạn sẽ thấy dòng chữ phản hồi từ API: *"Hello from Spring Boot! Ready for Digital Ocean deployment."*
