Giai đoạn 1: Chuẩn Bị & Đóng Gói (Trên máy cục bộ)
1️⃣ Phát triển ứng dụng Spring Boot:
✅ Sử dụng Gradle hoặc Maven để quản lý dự án.

✅ Cấu hình application.properties:

properties
Copy
Edit
server.port=8080
# Các biến DB sẽ được ghi đè bằng ENV trong Docker Compose
✅ Viết các lớp Controller, Service, Repository.

2️⃣ Tạo Dockerfile:
Dockerfile
Copy
Edit
FROM eclipse-temurin:21-jdk-alpine  
WORKDIR /app  
COPY build/libs/*.jar app.jar  
EXPOSE 8080  
CMD ["java", "-jar", "app.jar"]
3️⃣ Tạo docker-compose.yml:
yaml
Copy
Edit
version: "3.9"
services:
  db:
    image: postgres:16-alpine
    ports:
      - "5435:5432"
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass
      POSTGRES_DB: todos
    volumes:
      - pgdata:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U user"]
      interval: 10s
      retries: 5

  app:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "8080:8080"
    depends_on:
      db:
        condition: service_healthy

volumes:
  pgdata:
4️⃣ Build và Push (tùy chọn):
bash
Copy
Edit
./gradlew clean build
docker build -t asia-docker.pkg.dev/your-project-id/todo-api:latest .
docker push asia-docker.pkg.dev/your-project-id/todo-api:latest
☁️ Giai đoạn 2: Triển Khai (Trên GCP VM)
1️⃣ Tạo VM Instance trên Google Compute Engine:
Chọn Ubuntu hoặc Debian.

Cho phép HTTP/HTTPS nếu cần.

2️⃣ Cài đặt Docker & Docker Compose trên VM:
bash
Copy
Edit
sudo apt update && sudo apt install docker.io docker-compose -y
sudo systemctl start docker
3️⃣ Tải project lên VM:
Bạn có thể:

SFTP toàn bộ project từ máy cục bộ.

Hoặc clone từ GitHub (nếu đã push).

Hoặc chỉ cần docker-compose.yml nếu dùng image trên Artifact Registry.

4️⃣ Cấu hình Firewall trên GCP:
Mở cổng 8080 (ứng dụng Spring Boot).

(Tuỳ chọn) Mở cổng 5435 nếu cần kết nối DB từ bên ngoài.

5️⃣ Khởi động ứng dụng bằng Docker Compose:
bash
Copy
Edit
cd /path/to/project
docker compose up -d
⚙️ Giai đoạn 3: Vận Hành (Runtime)
🌐 Luồng Request từ Client:
Client gửi request đến VM IP:

arduino
Copy
Edit
http://YOUR_VM_IP:8080/api/todos
GCP Firewall cho phép nếu cổng 8080 đã mở.

Docker ánh xạ cổng 8080 trên VM → 8080 của container Spring Boot.

Ứng dụng Spring Boot nhận yêu cầu và xử lý.

🧠 Ứng dụng Spring Boot kết nối DB:
Dùng db:5432 vì:

Các container nằm trong Docker Network nội bộ (ví dụ: todo_default).

Docker hỗ trợ DNS nội bộ → tên db sẽ trỏ tới IP của container PostgreSQL.

🗃️ PostgreSQL xử lý truy vấn:
Nhận request SQL từ Spring Boot.

Đọc/Ghi dữ liệu vào: /var/lib/postgresql/data

Dữ liệu được gắn với volume pgdata → Lưu trên đĩa của VM.

💾 Lưu trữ bằng Docker Volume:
Volume pgdata đảm bảo tính bền vững:

Nếu container Postgres bị xoá, dữ liệu vẫn còn.

Volume tồn tại độc lập → dễ backup & restore.

🔁 Phản hồi về Client:
Spring Boot trả JSON (hoặc phản hồi phù hợp).

Trả qua container → cổng VM → về lại Client.

🔄 Tóm tắt Giao Tiếp Nội Bộ:
Thành phần	Mô tả
🧱 Docker Network	Tạo tự động bởi Docker Compose
🌐 Tên dịch vụ nội bộ	app, db dùng để giao tiếp
🔐 Port nội bộ	5432 chỉ dùng nội bộ giữa các container
📦 Volume pgdata	Gắn kết với container PostgreSQL để lưu trữ dữ liệu lâu dài
