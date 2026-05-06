# 🚀 Tổng quan flow
- SSH vào server
- Cài Java
- Pull code từ GitHub
- Build `.jar`
- Run thử app
- Setup Nginx reverse proxy
- Setup domain + SSL (Let's Encrypt)
- Tạo systemd service để auto chạy
- (Optional) CI/CD sau

## 🧩 Bước 1: SSH vào VPS
```bash
ssh root@your_server_ip
```

## ☕ Bước 2: Cài Java (OpenJDK 17)
Spring Boot thường dùng Java 17+
```bash
apt update
apt install openjdk-17-jdk -y
```
Check:
```bash
java -version
```

## 📦 Bước 3: Lấy source code
Nếu dùng GitHub:
```bash
apt install git -y
git clone https://github.com/your-repo.git
cd your-repo
```

## 🔨 Bước 4: Build project
Nếu dùng Maven:
```bash
chmod +x mvnw
./mvnw clean package -DskipTests
```
Nếu dùng Gradle:
```bash
./gradlew build
```
👉 Sau build sẽ có file:
`target/[name].jar`

## ▶️ Bước 5: Run thử app
```bash
java -jar target/[name].jar
```
Mặc định chạy port: `8080`

Test:
`http://your_server_ip:8080`

## 🌐 Bước 6: Cài Nginx
```bash
apt install nginx -y
```

## ⚙️ Bước 7: Config Nginx (reverse proxy)

### Tạo file config:
```bash
nano /etc/nginx/sites-available/app
```

```nginx
server {
   listen 80;
   server_name your_domain.com;

   location / {
       proxy_pass http://localhost:8080;
       proxy_set_header Host $host;
       proxy_set_header X-Real-IP $remote_addr;
   }
}
```

### Trỏ từ port => không port
🔎 Mở config Nginx mặc định:
```bash
nano /etc/nginx/sites-available/default
```

✏️ Sửa thành (QUAN TRỌNG):
```nginx
server {
   listen 80;
   server_name _;

   location / {
       proxy_pass http://localhost:8080;
       proxy_set_header Host $host;
       proxy_set_header X-Real-IP $remote_addr;
   }
}
```

### Enable & reload Nginx
```bash
ln -s /etc/nginx/sites-available/app /etc/nginx/sites-enabled/
nginx -t
systemctl restart nginx
```

## 🔒 Bước 8: SSL (HTTPS miễn phí)
Cài certbot:
```bash
apt install certbot python3-certbot-nginx -y
```
Chạy:
```bash
certbot --nginx -d domain.com
```
👉 Nó sẽ tự config HTTPS luôn

## 🔁 Bước 9: Tạo systemd service (QUAN TRỌNG)
Để app chạy background + auto restart

```bash
nano /etc/systemd/system/springboot.service
```

Thêm nội dung:
```ini
[Unit]
Description=Spring Boot App
After=network.target

[Service]
User=root
ExecStart=/usr/bin/java -jar /var/www/simple-spring-boot/target/digital-ocean-test-0.0.1-SNAPSHOT.jar
SuccessExitStatus=143
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

Reload và Start:
```bash
systemctl daemon-reexec
systemctl daemon-reload
systemctl start springboot
systemctl enable springboot
```

Check:
```bash
systemctl status springboot
```

## 📊 Bước 10: Debug & logs
```bash
journalctl -u springboot -f
```

## 💡 Bonus (rất nên làm)

### 🔥 Firewall
```bash
ufw allow OpenSSH
ufw allow 'Nginx Full'
ufw enable
```

### 🔄 Update app (deploy lại)
```bash
git pull
./mvnw package -DskipTests
systemctl restart springboot
```
