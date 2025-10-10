# บทที่ 9: Docker Commands และการ Host Web Application 🐳

## 🎯 สิ่งที่จะได้เรียนรู้
- คำสั่ง Docker ที่จำเป็นต้องรู้สำหรับการทำงานจริง
- วิธีเช็คการเชื่อมต่อระหว่าง Container กับ Host
- การ Deploy Web App ด้วย Docker Compose แบบมืออาชีพ
- การแก้ปัญหาที่เจอบ่อยๆ และวิธีรับมือ

## ⏰ เวลาที่ใช้: 90 นาที
- **0:00-0:20**: Docker Commands พื้นฐานที่ต้องรู้
- **0:20-0:40**: การเช็ค Connection และ Network
- **0:40-0:70**: Docker Compose สำหรับ Web Hosting
- **0:70-0:90**: Troubleshooting และ Best Practices

---

## 📚 Part 1: Docker Commands ที่ใช้งานจริงทุกวัน

### 1.1 คำสั่งพื้นฐานที่ต้องจำให้ขึ้นใจ 💪

```bash
# 🔍 ดู Container ที่กำลังทำงานอยู่
docker ps

# 📋 ดู Container ทั้งหมด (รวมที่หยุดแล้วด้วย)
docker ps -a

# 🖼️ ดู Images ทั้งหมด
docker images

# 🚀 Run Container แบบพื้นฐาน
docker run -d --name myapp -p 8080:80 nginx
# -d = ทำงานเบื้องหลัง (detached mode)
# --name = ตั้งชื่อ container
# -p = map port จาก host:container

# ⏹️ หยุด Container
docker stop myapp

# ▶️ Start Container ที่หยุดอยู่
docker start myapp

# 🔄 Restart Container
docker restart myapp

# 🗑️ ลบ Container (ต้องหยุดก่อน)
docker rm myapp

# 💥 ลบแบบบังคับ (Force remove - ใช้ตอนรีบๆ)
docker rm -f myapp
```

### 1.2 คำสั่งสำหรับการ Debug และแก้ปัญหา 🔧

```bash
# 📝 ดู Logs ของ Container
docker logs myapp

# 📡 ดู Logs แบบ Real-time (เหมือน tail -f)
docker logs -f myapp

# 🔬 ดู Logs ย้อนหลัง 100 บรรทัดล่าสุด
docker logs --tail 100 myapp

# 💻 เข้าไปใน Container (เหมือน SSH)
docker exec -it myapp bash
# หรือถ้าไม่มี bash ให้ใช้ sh
docker exec -it myapp sh

# 📊 ดูการใช้ Resources (CPU, Memory)
docker stats

# 🔍 ดูรายละเอียดของ Container
docker inspect myapp

# 📁 Copy ไฟล์จาก Container ออกมา
docker cp myapp:/etc/nginx/nginx.conf ./nginx.conf

# 📤 Copy ไฟล์เข้าไปใน Container
docker cp ./myfile.txt myapp:/tmp/
```

### 1.3 คำสั่งจัดการ Images 📦

```bash
# 🔽 ดาวน์โหลด Image จาก Docker Hub
docker pull node:18-alpine

# 🏗️ Build Image จาก Dockerfile
docker build -t myapp:v1 .

# 🏷️ ตั้งชื่อ/Tag ใหม่ให้ Image
docker tag myapp:v1 myapp:latest

# 📤 Push Image ขึ้น Registry
docker push myregistry.com/myapp:v1

# 🧹 ลบ Image
docker rmi myapp:v1

# 🧽 ลบ Images ที่ไม่ได้ใช้ทั้งหมด
docker image prune -a
```

---

## 🌐 Part 2: เช็ค Connection ระหว่าง Container กับ Host

### 2.1 เช็ค Network และ Port Mapping 🔌

```bash
# 📡 ดู Network ทั้งหมด
docker network ls

# 🔍 ดูรายละเอียด Network
docker network inspect bridge

# 🆕 สร้าง Network ใหม่
docker network create mynetwork

# 🔗 เชื่อม Container เข้ากับ Network
docker network connect mynetwork myapp

# ✂️ ตัดการเชื่อมต่อ
docker network disconnect mynetwork myapp
```

### 2.2 ทดสอบการเชื่อมต่อจริง 🧪

```bash
# 🌍 เช็คว่า Container เข้าถึง Internet ได้ไหม
docker exec myapp ping google.com

# 🏠 เช็คว่า Container เชื่อมต่อกับ Host ได้ไหม
docker exec myapp ping host.docker.internal

# 🔄 เช็คว่า Container คุยกันได้ไหม
docker exec container1 ping container2

# 🔍 ดู Port ที่ Container เปิดอยู่
docker port myapp

# 📊 เช็ค Port จาก Host
netstat -tulpn | grep 8080
# หรือใช้
ss -tulpn | grep 8080

# 🧪 ทดสอบเรียก API จากภายนอก
curl http://localhost:8080
```

### 2.3 วิธีแก้ปัญหา Connection ที่พบบ่อย 🚑

```bash
# ❌ ปัญหา: เข้า Container จาก Host ไม่ได้
# ✅ แก้ไข: เช็ค Port Mapping
docker run -d -p 0.0.0.0:8080:80 nginx
# 0.0.0.0 = รับ connection จากทุก interface

# ❌ ปัญหา: Container คุยกันไม่ได้
# ✅ แก้ไข: ใส่ Container ใน Network เดียวกัน
docker network create app-network
docker run -d --name backend --network app-network backend-image
docker run -d --name frontend --network app-network frontend-image

# ❌ ปัญหา: DNS ไม่ทำงาน
# ✅ แก้ไข: ใช้ --dns option
docker run -d --dns 8.8.8.8 --dns 8.8.4.4 myapp
```

---

## 🚀 Part 3: Docker Compose สำหรับ Web Hosting

### 3.1 โครงสร้างโปรเจค Web App พื้นฐาน 📁

```
my-web-app/
├── docker-compose.yml
├── .env
├── frontend/
│   ├── Dockerfile
│   └── src/
├── backend/
│   ├── Dockerfile
│   └── src/
└── nginx/
    └── default.conf
```

### 3.2 ตัวอย่าง docker-compose.yml สำหรับ Full-Stack App 📝

```yaml
version: '3.8'

services:
  # 🎨 Frontend Service (React/Vue/Angular)
  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile
    container_name: frontend-app
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=development
      - REACT_APP_API_URL=http://backend:5000
    volumes:
      - ./frontend:/app
      - /app/node_modules
    networks:
      - webnet
    depends_on:
      - backend

  # ⚙️ Backend Service (Node.js/Express)
  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile
    container_name: backend-api
    ports:
      - "5000:5000"
    environment:
      - NODE_ENV=development
      - DB_HOST=database
      - DB_PORT=5432
      - DB_NAME=${DB_NAME}
      - DB_USER=${DB_USER}
      - DB_PASSWORD=${DB_PASSWORD}
    volumes:
      - ./backend:/app
      - /app/node_modules
    networks:
      - webnet
    depends_on:
      - database

  # 🗄️ Database Service (PostgreSQL)
  database:
    image: postgres:15-alpine
    container_name: postgres-db
    ports:
      - "5432:5432"
    environment:
      - POSTGRES_DB=${DB_NAME}
      - POSTGRES_USER=${DB_USER}
      - POSTGRES_PASSWORD=${DB_PASSWORD}
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql
    networks:
      - webnet

  # 🌐 Nginx Reverse Proxy
  nginx:
    image: nginx:alpine
    container_name: nginx-proxy
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/default.conf:/etc/nginx/conf.d/default.conf
      - ./ssl:/etc/nginx/ssl
    networks:
      - webnet
    depends_on:
      - frontend
      - backend

  # 📊 Adminer (Database Management)
  adminer:
    image: adminer
    container_name: adminer
    ports:
      - "8080:8080"
    networks:
      - webnet
    depends_on:
      - database

networks:
  webnet:
    driver: bridge

volumes:
  postgres_data:
```

### 3.3 ไฟล์ .env สำหรับเก็บ Secrets 🔐

```bash
# Database Configuration
DB_NAME=myapp_db
DB_USER=myapp_user
DB_PASSWORD=supersecretpassword123

# App Configuration
APP_PORT=3000
API_PORT=5000

# Environment
NODE_ENV=development
```

### 3.4 Nginx Configuration สำหรับ Reverse Proxy 🔧

```nginx
# nginx/default.conf
upstream frontend {
    server frontend:3000;
}

upstream backend {
    server backend:5000;
}

server {
    listen 80;
    server_name localhost;

    # Frontend Routes
    location / {
        proxy_pass http://frontend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # Backend API Routes
    location /api {
        rewrite ^/api(.*)$ $1 break;
        proxy_pass http://backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # WebSocket Support
    location /ws {
        proxy_pass http://backend;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
```

---

## 🎮 Part 4: คำสั่ง Docker Compose ที่ใช้งานจริง

### 4.1 คำสั่งพื้นฐาน 🏃‍♂️

```bash
# 🚀 Start ทุก Service
docker-compose up

# 🌙 Start แบบ Background (Detached)
docker-compose up -d

# 🏗️ Build และ Start
docker-compose up -d --build

# ⏹️ หยุดทุก Service
docker-compose stop

# 🔻 หยุดและลบ Container/Network
docker-compose down

# 💥 ลบทุกอย่างรวม Volumes
docker-compose down -v

# 📋 ดู Service ที่กำลังทำงาน
docker-compose ps

# 📝 ดู Logs ทั้งหมด
docker-compose logs

# 🔍 ดู Logs เฉพาะ Service
docker-compose logs backend

# 📡 ดู Logs แบบ Real-time
docker-compose logs -f backend
```

### 4.2 คำสั่งขั้นสูง 🚁

```bash
# 🔄 Restart Service เฉพาะตัว
docker-compose restart backend

# 📈 Scale Service (เพิ่มจำนวน Instance)
docker-compose up -d --scale backend=3

# 🔧 Execute คำสั่งใน Service
docker-compose exec backend bash

# 🏃 Run คำสั่งแบบ One-off
docker-compose run backend npm test

# 🔍 Validate docker-compose.yml
docker-compose config

# 🌊 ดู Events แบบ Real-time
docker-compose events

# 📊 ดูการใช้ Resources
docker-compose top
```

---

## 🛠️ Part 5: Workflow การ Deploy Web App

### 5.1 ขั้นตอนการ Setup และ Deploy 📋

```bash
# Step 1: Clone โปรเจค
git clone https://github.com/yourname/your-web-app.git
cd your-web-app

# Step 2: สร้างไฟล์ .env
cp .env.example .env
# แก้ไขค่าต่างๆ ใน .env

# Step 3: Build Images
docker-compose build

# Step 4: Start Services
docker-compose up -d

# Step 5: Check Status
docker-compose ps
docker-compose logs

# Step 6: Initialize Database (ถ้าจำเป็น)
docker-compose exec backend npm run migrate
docker-compose exec backend npm run seed

# Step 7: ทดสอบการทำงาน
curl http://localhost
curl http://localhost/api/health
```

### 5.2 การ Update และ Deploy ใหม่ 🔄

```bash
# 📥 Pull โค้ดใหม่
git pull origin main

# 🏗️ Build Image ใหม่
docker-compose build backend

# 🔄 Rolling Update (ไม่มี Downtime)
docker-compose up -d --no-deps --build backend

# 🧹 Clean up Images เก่า
docker image prune -f
```

### 5.3 Backup และ Restore Database 💾

```bash
# 💾 Backup Database
docker-compose exec database pg_dump -U myapp_user myapp_db > backup.sql

# 📥 Restore Database
docker-compose exec -T database psql -U myapp_user myapp_db < backup.sql

# 🔄 Backup Volumes
docker run --rm -v myapp_postgres_data:/data -v $(pwd):/backup alpine tar czf /backup/db_backup.tar.gz /data

# 📤 Restore Volumes
docker run --rm -v myapp_postgres_data:/data -v $(pwd):/backup alpine tar xzf /backup/db_backup.tar.gz -C /
```

---

## 🚨 Part 6: Troubleshooting ปัญหาที่เจอบ่อย

### 6.1 Port ชนกัน 🔴

```bash
# ❌ Error: bind: address already in use
# ✅ แก้ไข: หา Process ที่ใช้ Port อยู่
lsof -i :8080
# หรือ
netstat -tulpn | grep 8080

# Kill Process ที่ใช้ Port
kill -9 <PID>

# หรือเปลี่ยน Port ใน docker-compose.yml
ports:
  - "8081:8080"  # เปลี่ยนจาก 8080 เป็น 8081
```

### 6.2 Container Exit ทันที 💥

```bash
# 🔍 ดู Exit Code
docker-compose ps

# 📝 ดู Logs เพื่อหาสาเหตุ
docker-compose logs backend

# 🔧 Debug โดยเข้าไปดูใน Container
docker-compose run --rm backend sh
# ลองรันคำสั่ง start ดูว่า error อะไร
```

### 6.3 ปัญหา Permission 🔐

```bash
# ❌ Error: Permission denied
# ✅ แก้ไข: ตั้งค่า User ใน Dockerfile
# Dockerfile
USER node
WORKDIR /app
COPY --chown=node:node . .

# หรือใน docker-compose.yml
user: "1000:1000"  # UID:GID ของ user
```

### 6.4 Container หา Service อื่นไม่เจอ 🔍

```bash
# ❌ Error: ECONNREFUSED หรือ Connection refused
# ✅ แก้ไข: ใช้ชื่อ Service แทน localhost

# ❌ ผิด
const dbHost = 'localhost';
const apiUrl = 'http://127.0.0.1:5000';

# ✅ ถูก
const dbHost = 'database';  # ชื่อ service ใน docker-compose
const apiUrl = 'http://backend:5000';
```

---

## 📊 Part 7: Monitoring และ Performance

### 7.1 เช็คการใช้ Resources 📈

```bash
# 📊 ดู CPU/Memory แบบ Real-time
docker stats

# 🔍 ดูเฉพาะบาง Container
docker stats frontend backend database

# 📋 ดูแบบไม่ Real-time (ครั้งเดียว)
docker stats --no-stream

# 💾 เช็คขนาด Disk ที่ Docker ใช้
docker system df

# 🧹 ทำความสะอาด Space
docker system prune -a --volumes
```

### 7.2 Health Check 🏥

```yaml
# ใน docker-compose.yml
services:
  backend:
    # ... config อื่นๆ ...
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:5000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
```

```bash
# 🏥 เช็ค Health Status
docker-compose ps
# ดูคอลัมน์ State จะเห็น (healthy) หรือ (unhealthy)

# 🔍 ดูรายละเอียด Health Check
docker inspect backend | grep -A 10 Health
```

---

## 🎯 Best Practices สำหรับ Production

### 1. ใช้ Specific Version ของ Image 🏷️
```yaml
# ❌ ไม่ดี
image: node:latest

# ✅ ดี
image: node:18.17-alpine3.18
```

### 2. ตั้งค่า Resource Limits 📏
```yaml
services:
  backend:
    deploy:
      resources:
        limits:
          cpus: '1.0'
          memory: 512M
        reservations:
          cpus: '0.5'
          memory: 256M
```

### 3. ใช้ .dockerignore 📝
```bash
# .dockerignore
node_modules
.git
.env
*.log
.DS_Store
coverage
.nyc_output
```

### 4. Multi-stage Build สำหรับ Production 🏗️
```dockerfile
# Build Stage
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

# Production Stage
FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/node_modules ./node_modules
COPY . .
EXPOSE 3000
CMD ["node", "server.js"]
```

### 5. ใช้ Docker Secrets สำหรับข้อมูลสำคัญ 🔐
```yaml
services:
  backend:
    secrets:
      - db_password
    environment:
      DB_PASSWORD_FILE: /run/secrets/db_password

secrets:
  db_password:
    file: ./secrets/db_password.txt
```

---

## 📝 สรุป Cheat Sheet

### คำสั่งที่ใช้บ่อยที่สุด Top 10 🏆

```bash
# 1. Start ทุกอย่าง
docker-compose up -d

# 2. ดู Logs
docker-compose logs -f

# 3. เข้าไปใน Container
docker-compose exec backend bash

# 4. Restart Service
docker-compose restart backend

# 5. หยุดทุกอย่าง
docker-compose down

# 6. Build ใหม่
docker-compose up -d --build

# 7. ดู Status
docker-compose ps

# 8. ลบทุกอย่าง
docker-compose down -v

# 9. ดูการใช้ Resources
docker stats

# 10. Clean up
docker system prune -a
```

---

## 🎬 Workshop: Deploy Full-Stack App

### เวลา 20 นาที - ลองทำจริง! 💪

1. **Setup Project Structure**
```bash
mkdir my-fullstack-app
cd my-fullstack-app
mkdir frontend backend nginx
```

2. **Create docker-compose.yml**
```bash
# Copy docker-compose.yml จากตัวอย่างด้านบน
```

3. **Create Simple Backend** (backend/index.js)
```javascript
const express = require('express');
const app = express();

app.get('/api/health', (req, res) => {
  res.json({ status: 'OK', timestamp: new Date() });
});

app.listen(5000, () => {
  console.log('Backend running on port 5000');
});
```

4. **Create Simple Frontend** (frontend/index.html)
```html
<!DOCTYPE html>
<html>
<head>
    <title>My App</title>
</head>
<body>
    <h1>Hello Docker Compose!</h1>
    <div id="health"></div>
    <script>
        fetch('/api/health')
            .then(res => res.json())
            .then(data => {
                document.getElementById('health').innerHTML =
                    `API Status: ${data.status}`;
            });
    </script>
</body>
</html>
```

5. **Deploy!**
```bash
docker-compose up -d
# เปิด http://localhost ดูผลงาน!
```

---

## 🎉 จบแล้ว! คุณพร้อมใช้ Docker ในการทำงานจริงแล้ว

### 💡 Tips สุดท้าย
- อย่าลืม commit docker-compose.yml เข้า Git
- ใช้ .env.example เป็นตัวอย่างสำหรับทีม
- ทำ Documentation ของ Services ต่างๆ
- Setup CI/CD pipeline สำหรับ Auto Deploy

### 📚 ศึกษาเพิ่มเติม
- Docker Swarm สำหรับ Production Cluster
- Kubernetes สำหรับ Enterprise Scale
- Monitoring ด้วย Prometheus + Grafana
- Log Management ด้วย ELK Stack

**Happy Dockering! 🐳✨**