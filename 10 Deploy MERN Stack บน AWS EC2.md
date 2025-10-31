# บทที่ 10: Deploy MERN Stack Application บน AWS EC2 ด้วย Docker Compose

## 🎯 เป้าหมายการเรียนรู้
- สร้าง Virtual Machine บน AWS EC2
- ติดตั้งและกำหนดค่า Ubuntu Server
- ติดตั้ง Docker และ Docker Compose บน EC2
- Deploy MERN Stack Application ด้วย Nginx
- เข้าใจ CI/CD และ Continuous Delivery

## ⏰ เวลาที่ใช้: 90 นาที
- **0:00-0:20**: สร้าง EC2 Instance และเชื่อมต่อ
- **0:20-0:40**: ติดตั้ง Docker, Docker Compose และ Nginx
- **0:40-0:70**: Deploy MERN Stack Application
- **0:70-0:90**: ทดสอบและ Troubleshooting

---

## 📚 ความรู้พื้นฐาน: CI/CD คืออะไร?

### Continuous Integration (CI)
การรวม Feature ใหม่เข้าไปในระบบอย่างต่อเนื่องโดยไม่หยุดให้บริการ
- Automated testing
- Automated building
- Quick feedback loop

### Continuous Delivery (CD)
การให้บริการอย่างต่อเนื่องไม่หยุดชะงัก
- Automated deployment
- Zero downtime updates
- Rollback capabilities

### เส้นทางการ Deploy ที่เรียนมา
```
Localhost Development
    ↓
Docker Containerization
    ↓
Vercel (Static Sites) → CI/CD
    ↓
GitHub Actions
    ↓
AWS EC2 (Full-Stack) ← เราอยู่ตรงนี้!
```

---

## 🌟 ส่วนที่ 1: สร้าง AWS EC2 Instance

### 1.1 Prerequisites
- บัญชี AWS (Free Tier)
- Terminal หรือ PowerShell
- Key pair สำหรับ SSH

### 1.2 ขั้นตอนการสร้าง EC2 Instance

#### Step 1: เข้า AWS Console และเลือก EC2
1. ไปที่ https://console.aws.amazon.com
2. เลือก Region (แนะนำ: Singapore ap-southeast-1)
3. ไปที่ Services → EC2
4. คลิก **Launch Instance**

#### Step 2: กำหนดค่า Instance

**a) ชื่อและ OS (Operating System)**
```
Name: mern-stack-server
Application and OS Images (AMI):
  → Ubuntu Server 22.04 LTS (HVM), SSD Volume Type
  → 64-bit (x86)
```

**b) Instance Type (Hardware)**
```
Instance type: t2.micro (Free tier eligible)
  - 1 vCPU
  - 1 GB RAM

หรือหากต้องการประสิทธิภาพดีกว่า:
  t3.small (ไม่ฟรี)
  - 2 vCPU
  - 2 GB RAM
```

**c) Key Pair สำหรับ SSH**
```
1. Create new key pair
   Name: mern-stack-key
   Type: RSA
   Format: .pem (สำหรับ Mac/Linux/WSL)
          .ppk (สำหรับ PuTTY บน Windows)

2. Download key pair file
3. เก็บไว้ที่ ~/.ssh/
```

**วิธีจัดเก็บ Key Pair:**
```bash
# บน Mac/Linux/WSL
mkdir -p ~/.ssh
mv ~/Downloads/mern-stack-key.pem ~/.ssh/
chmod 400 ~/.ssh/mern-stack-key.pem
```

```powershell
# บน Windows PowerShell
mkdir $HOME\.ssh
move $HOME\Downloads\mern-stack-key.pem $HOME\.ssh\
```

**d) Network Settings (Security Group)**
```
Create security group: Yes

Security group name: mern-stack-sg

Inbound rules:
┌──────────────────┬──────┬─────────────┬─────────────────┐
│ Type             │ Port │ Source      │ Description     │
├──────────────────┼──────┼─────────────┼─────────────────┤
│ SSH              │ 22   │ My IP       │ SSH access      │
│ HTTP             │ 80   │ 0.0.0.0/0   │ Web traffic     │
│ HTTPS            │ 443  │ 0.0.0.0/0   │ Secure web      │
│ Custom TCP       │ 3000 │ 0.0.0.0/0   │ Frontend dev    │
│ Custom TCP       │ 5000 │ 0.0.0.0/0   │ Backend API     │
└──────────────────┴──────┴─────────────┴─────────────────┘

⚠️ หมายเหตุ: ใน Production ไม่ควรเปิด port 3000, 5000 ให้ public
ควรใช้ Nginx เป็น reverse proxy แทน
```

**e) Storage (Hard Disk)**
```
Configure storage:
  Volume 1 (Root):
    Size: 20 GB
    Volume type: gp3 (General Purpose SSD)
    Delete on termination: Yes
```

#### Step 3: Launch Instance
```
1. Review settings
2. คลิก Launch Instance
3. รอ Instance State: Running (ประมาณ 1-2 นาที)
```

---

## 🔌 ส่วนที่ 2: เชื่อมต่อกับ EC2 Instance

### 2.1 หา Public IP Address

```
EC2 Dashboard → Instances → เลือก instance ของคุณ
จะเห็น:
  - Instance ID: i-0123456789abcdef0
  - Public IPv4 address: 13.250.123.45 (ตัวอย่าง)
  - Public IPv4 DNS: ec2-13-250-123-45.ap-southeast-1.compute.amazonaws.com
```

### 2.2 เชื่อมต่อผ่าน SSH

#### บน Mac/Linux/WSL:
```bash
# เปลี่ยน [YOUR-IP] เป็น Public IP จริงของคุณ
ssh -i ~/.ssh/mern-stack-key.pem ubuntu@[YOUR-IP]

# ตัวอย่าง
ssh -i ~/.ssh/mern-stack-key.pem ubuntu@13.250.123.45
```

#### บน Windows PowerShell:
```powershell
ssh -i $HOME\.ssh\mern-stack-key.pem ubuntu@[YOUR-IP]
```

#### การแก้ปัญหา Permission (ถ้ามี error)
```bash
# Mac/Linux
chmod 400 ~/.ssh/mern-stack-key.pem

# Windows (PowerShell as Admin)
icacls $HOME\.ssh\mern-stack-key.pem /inheritance:r
icacls $HOME\.ssh\mern-stack-key.pem /grant:r "$($env:USERNAME):(R)"
```

### 2.3 ตรวจสอบการเชื่อมต่อ

เมื่อเข้าสู่ระบบสำเร็จ จะเห็น:
```
Welcome to Ubuntu 22.04.3 LTS
...
ubuntu@ip-172-31-x-x:~$
```

**User ที่ใช้งาน:**
- Default user: `ubuntu` (มีสิทธิ์ sudo)
- เป็น root user (ใช้ sudo ได้โดยไม่ต้องใส่รหัสผ่าน)

---

## 🔧 ส่วนที่ 3: เตรียม Ubuntu Server

### 3.1 อัพเดตระบบ

```bash
# เช็คว่ามีแพ็คเกจอะไรต้องอัพเดตบ้าง
sudo apt update

# อัพเกรดแพ็คเกจทั้งหมด
sudo apt upgrade -y

# รีบูตระบบ (ถ้าจำเป็น - เช่นมีการอัพเดต kernel)
sudo reboot
# หรือ
sudo init 6
```

หลัง reboot ให้ SSH เข้าไปใหม่:
```bash
ssh -i ~/.ssh/mern-stack-key.pem ubuntu@[YOUR-IP]
```

### 3.2 ติดตั้ง Packages พื้นฐาน

```bash
# ติดตั้ง tools ที่จำเป็น
sudo apt install -y \
  curl \
  wget \
  git \
  vim \
  htop \
  net-tools
```

---

## 🐳 ส่วนที่ 4: ติดตั้ง Docker และ Docker Compose

### 4.1 ติดตั้ง Docker

```bash
# ลบ Docker เวอร์ชันเก่า (ถ้ามี)
sudo apt remove docker docker-engine docker.io containerd runc

# ติดตั้ง dependencies
sudo apt update
sudo apt install -y \
  ca-certificates \
  curl \
  gnupg \
  lsb-release

# เพิ่ม Docker's official GPG key
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
  sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# เพิ่ม Docker repository
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# ติดตั้ง Docker Engine
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin

# เพิ่ม user ubuntu เข้า docker group (ไม่ต้องใช้ sudo)
sudo usermod -aG docker ubuntu

# Logout และ login ใหม่เพื่อให้ group มีผล
exit
```

SSH เข้าไปใหม่:
```bash
ssh -i ~/.ssh/mern-stack-key.pem ubuntu@[YOUR-IP]
```

### 4.2 ทดสอบ Docker

```bash
# เช็ค Docker version
docker --version
# Output: Docker version 24.0.x, build xxxxx

# ทดสอบรัน container
docker run hello-world

# เช็ค Docker Compose
docker compose version
# Output: Docker Compose version v2.x.x
```

---

## 🌐 ส่วนที่ 5: ติดตั้ง Nginx

### 5.1 ติดตั้ง Nginx

```bash
# ติดตั้ง Nginx
sudo apt install -y nginx

# เช็คสถานะ
sudo systemctl status nginx

# Start Nginx (ถ้ายังไม่ทำงาน)
sudo systemctl start nginx

# ตั้งให้ start อัตโนมัติเมื่อ boot
sudo systemctl enable nginx
```

### 5.2 ทดสอบ Nginx

เปิด browser ไปที่:
```
http://[YOUR-IP]
```

ควรเห็นหน้า **Welcome to nginx!**

### 5.3 กำหนดค่า Firewall (UFW)

```bash
# เปิดใช้งาน UFW
sudo ufw enable

# อนุญาต SSH (สำคัญมาก! ไม่งั้นจะ SSH ไม่ได้)
sudo ufw allow ssh
sudo ufw allow 22/tcp

# อนุญาต HTTP และ HTTPS
sudo ufw allow 'Nginx Full'
# หรือ
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# เช็คสถานะ
sudo ufw status
```

Output ที่ควรเห็น:
```
Status: active

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW       Anywhere
80/tcp                     ALLOW       Anywhere
443/tcp                    ALLOW       Anywhere
```

---

## 🚀 ส่วนที่ 6: Deploy MERN Stack Application

### 6.1 เตรียมโครงสร้างโปรเจกต์

```bash
# สร้าง directory สำหรับ project
mkdir -p ~/mern-app
cd ~/mern-app

# สร้างโครงสร้างโฟลเดอร์
mkdir -p frontend backend nginx
```

### 6.2 สร้าง Backend (Express + MongoDB)

#### สร้าง Backend Dockerfile
```bash
cd ~/mern-app/backend
nano Dockerfile
```

**backend/Dockerfile:**
```dockerfile
FROM node:18-alpine

WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm ci --only=production

# Copy application code
COPY . .

EXPOSE 5000

CMD ["node", "server.js"]
```

#### สร้าง Backend Code
```bash
nano package.json
```

**backend/package.json:**
```json
{
  "name": "mern-backend",
  "version": "1.0.0",
  "main": "server.js",
  "scripts": {
    "start": "node server.js"
  },
  "dependencies": {
    "express": "^4.18.2",
    "mongoose": "^8.0.0",
    "cors": "^2.8.5",
    "dotenv": "^16.3.1"
  }
}
```

```bash
nano server.js
```

**backend/server.js:**
```javascript
const express = require('express');
const mongoose = require('mongoose');
const cors = require('cors');
require('dotenv').config();

const app = express();
const PORT = process.env.PORT || 5000;

// Middleware
app.use(cors());
app.use(express.json());

// MongoDB Connection
mongoose.connect(process.env.MONGODB_URI || 'mongodb://mongo:27017/merndb')
  .then(() => console.log('✅ MongoDB connected'))
  .catch(err => console.error('❌ MongoDB connection error:', err));

// Simple Schema
const ItemSchema = new mongoose.Schema({
  name: String,
  description: String,
  createdAt: { type: Date, default: Date.now }
});

const Item = mongoose.model('Item', ItemSchema);

// Routes
app.get('/api/health', (req, res) => {
  res.json({
    status: 'OK',
    message: 'Backend is running!',
    timestamp: new Date()
  });
});

app.get('/api/items', async (req, res) => {
  try {
    const items = await Item.find().sort({ createdAt: -1 });
    res.json(items);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

app.post('/api/items', async (req, res) => {
  try {
    const newItem = new Item(req.body);
    const savedItem = await newItem.save();
    res.status(201).json(savedItem);
  } catch (error) {
    res.status(400).json({ error: error.message });
  }
});

app.delete('/api/items/:id', async (req, res) => {
  try {
    await Item.findByIdAndDelete(req.params.id);
    res.json({ message: 'Item deleted' });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

app.listen(PORT, '0.0.0.0', () => {
  console.log(`🚀 Backend running on port ${PORT}`);
});
```

```bash
nano .env
```

**backend/.env:**
```env
PORT=5000
MONGODB_URI=mongodb://mongo:27017/merndb
NODE_ENV=production
```

### 6.3 สร้าง Frontend (React)

```bash
cd ~/mern-app/frontend
nano Dockerfile
```

**frontend/Dockerfile:**
```dockerfile
# Build stage
FROM node:18-alpine AS builder

WORKDIR /app

COPY package*.json ./
RUN npm ci

COPY . .
RUN npm run build

# Production stage
FROM nginx:alpine

COPY --from=builder /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

**frontend/package.json:**
```bash
nano package.json
```

```json
{
  "name": "mern-frontend",
  "version": "1.0.0",
  "type": "module",
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview"
  },
  "dependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0"
  },
  "devDependencies": {
    "@vitejs/plugin-react": "^4.2.0",
    "vite": "^5.0.0"
  }
}
```

**frontend/index.html:**
```bash
nano index.html
```

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>MERN Stack on AWS</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      max-width: 800px;
      margin: 50px auto;
      padding: 20px;
      background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
      color: white;
    }
    .container {
      background: white;
      color: #333;
      padding: 30px;
      border-radius: 10px;
      box-shadow: 0 10px 30px rgba(0,0,0,0.3);
    }
    h1 { color: #667eea; }
    input, button {
      padding: 10px;
      margin: 10px 0;
      border-radius: 5px;
      border: 1px solid #ddd;
    }
    button {
      background: #667eea;
      color: white;
      border: none;
      cursor: pointer;
    }
    button:hover { background: #5568d3; }
    .item {
      background: #f5f5f5;
      padding: 15px;
      margin: 10px 0;
      border-radius: 5px;
      display: flex;
      justify-content: space-between;
      align-items: center;
    }
    .delete-btn {
      background: #dc3545;
      padding: 5px 15px;
      font-size: 14px;
    }
    .status {
      padding: 10px;
      border-radius: 5px;
      margin: 10px 0;
    }
    .status.ok { background: #d4edda; color: #155724; }
    .status.error { background: #f8d7da; color: #721c24; }
  </style>
</head>
<body>
  <div class="container">
    <h1>🚀 MERN Stack on AWS EC2</h1>
    <div id="status"></div>

    <div>
      <h2>Add New Item</h2>
      <input type="text" id="name" placeholder="Item name" />
      <input type="text" id="description" placeholder="Description" />
      <button onclick="addItem()">Add Item</button>
    </div>

    <div>
      <h2>Items List</h2>
      <button onclick="loadItems()">Refresh</button>
      <div id="items"></div>
    </div>
  </div>

  <script>
    const API_URL = window.location.hostname === 'localhost'
      ? 'http://localhost:5000/api'
      : '/api';

    async function checkBackend() {
      try {
        const res = await fetch(`${API_URL}/health`);
        const data = await res.json();
        document.getElementById('status').innerHTML =
          `<div class="status ok">✅ Backend: ${data.message}</div>`;
      } catch (error) {
        document.getElementById('status').innerHTML =
          `<div class="status error">❌ Backend connection failed</div>`;
      }
    }

    async function loadItems() {
      try {
        const res = await fetch(`${API_URL}/items`);
        const items = await res.json();

        const itemsHtml = items.map(item => `
          <div class="item">
            <div>
              <strong>${item.name}</strong><br>
              <small>${item.description}</small><br>
              <small>Created: ${new Date(item.createdAt).toLocaleString()}</small>
            </div>
            <button class="delete-btn" onclick="deleteItem('${item._id}')">Delete</button>
          </div>
        `).join('');

        document.getElementById('items').innerHTML = itemsHtml || '<p>No items yet</p>';
      } catch (error) {
        console.error('Error loading items:', error);
      }
    }

    async function addItem() {
      const name = document.getElementById('name').value;
      const description = document.getElementById('description').value;

      if (!name || !description) {
        alert('Please fill all fields');
        return;
      }

      try {
        await fetch(`${API_URL}/items`, {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify({ name, description })
        });

        document.getElementById('name').value = '';
        document.getElementById('description').value = '';
        loadItems();
      } catch (error) {
        console.error('Error adding item:', error);
      }
    }

    async function deleteItem(id) {
      try {
        await fetch(`${API_URL}/items/${id}`, { method: 'DELETE' });
        loadItems();
      } catch (error) {
        console.error('Error deleting item:', error);
      }
    }

    // Load data on page load
    checkBackend();
    loadItems();
  </script>
</body>
</html>
```

**frontend/vite.config.js:**
```bash
nano vite.config.js
```

```javascript
import { defineConfig } from 'vite'

export default defineConfig({
  server: {
    port: 3000,
    host: true
  },
  build: {
    outDir: 'dist'
  }
})
```

**frontend/nginx.conf:**
```bash
nano nginx.conf
```

```nginx
server {
    listen 80;
    server_name _;

    root /usr/share/nginx/html;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }

    # API proxy
    location /api {
        proxy_pass http://backend:5000/api;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

### 6.4 สร้าง Nginx Reverse Proxy Configuration

```bash
cd ~/mern-app/nginx
nano default.conf
```

**nginx/default.conf:**
```nginx
upstream frontend {
    server frontend:80;
}

upstream backend {
    server backend:5000;
}

server {
    listen 80;
    server_name _;

    # Frontend
    location / {
        proxy_pass http://frontend;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # Backend API
    location /api {
        proxy_pass http://backend:5000/api;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### 6.5 สร้าง Docker Compose File

```bash
cd ~/mern-app
nano docker-compose.yml
```

**docker-compose.yml:**
```yaml
version: '3.8'

services:
  # MongoDB Database
  mongo:
    image: mongo:7-jammy
    container_name: mongodb
    restart: always
    environment:
      MONGO_INITDB_DATABASE: merndb
    volumes:
      - mongo_data:/data/db
    networks:
      - mern-network
    healthcheck:
      test: echo 'db.runCommand("ping").ok' | mongosh localhost:27017/test --quiet
      interval: 10s
      timeout: 5s
      retries: 5

  # Backend API
  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile
    container_name: backend-api
    restart: always
    environment:
      - NODE_ENV=production
      - PORT=5000
      - MONGODB_URI=mongodb://mongo:27017/merndb
    depends_on:
      mongo:
        condition: service_healthy
    networks:
      - mern-network

  # Frontend
  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile
    container_name: frontend-app
    restart: always
    depends_on:
      - backend
    networks:
      - mern-network

  # Nginx Reverse Proxy
  nginx:
    image: nginx:alpine
    container_name: nginx-proxy
    restart: always
    ports:
      - "80:80"
    volumes:
      - ./nginx/default.conf:/etc/nginx/conf.d/default.conf
    depends_on:
      - frontend
      - backend
    networks:
      - mern-network

volumes:
  mongo_data:

networks:
  mern-network:
    driver: bridge
```

### 6.6 Build และ Deploy!

```bash
# ตรวจสอบโครงสร้างไฟล์
cd ~/mern-app
tree -L 2
```

ควรเห็น:
```
mern-app/
├── docker-compose.yml
├── backend/
│   ├── Dockerfile
│   ├── package.json
│   ├── server.js
│   └── .env
├── frontend/
│   ├── Dockerfile
│   ├── package.json
│   ├── index.html
│   ├── vite.config.js
│   └── nginx.conf
└── nginx/
    └── default.conf
```

**Build และรัน:**
```bash
# Build images
docker compose build

# Start services
docker compose up -d

# เช็คสถานะ
docker compose ps

# ดู logs
docker compose logs -f
```

### 6.7 ทดสอบ Application

เปิด browser ไปที่:
```
http://[YOUR-EC2-IP]
```

ควรเห็นหน้า MERN Stack application!

**ทดสอบ API โดยตรง:**
```bash
curl http://localhost/api/health
curl http://localhost/api/items
```

---

## 🔍 ส่วนที่ 7: Monitoring และ Troubleshooting

### 7.1 คำสั่งสำหรับ Debug

```bash
# ดูสถานะ containers
docker compose ps

# ดู logs ทั้งหมด
docker compose logs

# ดู logs แบบ real-time
docker compose logs -f

# ดู logs เฉพาะ service
docker compose logs backend
docker compose logs frontend
docker compose logs mongo

# เช็ค resources
docker stats

# เข้าไปใน container
docker compose exec backend sh
docker compose exec mongo mongosh
```

### 7.2 ปัญหาที่พบบ่อย

#### ❌ ปัญหา: Backend เชื่อมต่อ MongoDB ไม่ได้
```bash
# เช็ค MongoDB
docker compose exec mongo mongosh
> show dbs
> use merndb
> show collections

# Restart services
docker compose restart backend
```

#### ❌ ปัญหา: Port 80 ถูกใช้โดย Nginx default
```bash
# หยุด Nginx ที่ติดตั้งไว้
sudo systemctl stop nginx
sudo systemctl disable nginx

# Start Docker Compose ใหม่
docker compose up -d
```

#### ❌ ปัญหา: Out of Memory
```bash
# เพิ่ม Swap Space
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# ทำให้ persist หลัง reboot
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

#### ❌ ปัญหา: Disk Full
```bash
# เช็ค disk space
df -h

# ลบ Docker images/containers ที่ไม่ใช้
docker system prune -a --volumes

# ลบ logs เก่า
sudo journalctl --vacuum-time=3d
```

### 7.3 Health Check Script

```bash
cd ~/mern-app
nano healthcheck.sh
```

```bash
#!/bin/bash

echo "🔍 MERN Stack Health Check"
echo "=========================="

# Check Docker
echo "📦 Docker Status:"
docker compose ps

# Check Backend
echo -e "\n🔧 Backend Health:"
curl -s http://localhost/api/health | jq

# Check MongoDB
echo -e "\n🗄️  MongoDB Status:"
docker compose exec -T mongo mongosh --eval "db.adminCommand('ping')" --quiet

# Check Disk Space
echo -e "\n💾 Disk Space:"
df -h | grep -E 'Filesystem|/$'

# Check Memory
echo -e "\n🧠 Memory Usage:"
free -h

echo -e "\n✅ Health check completed!"
```

```bash
chmod +x healthcheck.sh
./healthcheck.sh
```

---

## 🚦 ส่วนที่ 8: การจัดการ Application

### 8.1 คำสั่งที่ใช้บ่อย

```bash
# Start application
docker compose up -d

# Stop application
docker compose stop

# Restart application
docker compose restart

# Stop และลบทุกอย่าง
docker compose down

# Stop และลบทุกอย่างรวม volumes (ข้อมูลจะหาย!)
docker compose down -v

# Update code และ rebuild
git pull  # ถ้าใช้ git
docker compose build
docker compose up -d --no-deps --build [service-name]

# View logs
docker compose logs -f --tail=100

# Backup MongoDB
docker compose exec mongo mongodump --out=/tmp/backup
docker cp mongodb:/tmp/backup ./mongodb-backup-$(date +%Y%m%d)
```

### 8.2 การทำ Backup

```bash
# สร้าง backup script
nano ~/backup.sh
```

```bash
#!/bin/bash

BACKUP_DIR="$HOME/backups"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p $BACKUP_DIR

# Backup MongoDB
echo "Backing up MongoDB..."
docker compose exec -T mongo mongodump --archive --gzip > \
  $BACKUP_DIR/mongodb_$DATE.archive.gz

# Backup application code
echo "Backing up application code..."
tar -czf $BACKUP_DIR/app_$DATE.tar.gz \
  -C ~/mern-app \
  --exclude='node_modules' \
  --exclude='.git' \
  .

echo "✅ Backup completed: $BACKUP_DIR"
ls -lh $BACKUP_DIR | tail -5
```

```bash
chmod +x ~/backup.sh

# ทดสอบ backup
~/backup.sh

# ตั้ง cron job สำหรับ backup อัตโนมัติ (ทุกวันเวลา 2 AM)
crontab -e
# เพิ่มบรรทัดนี้:
# 0 2 * * * /home/ubuntu/backup.sh >> /home/ubuntu/backup.log 2>&1
```

---

## 🎓 ส่วนที่ 9: Best Practices

### 9.1 Security

```bash
# 1. ปรับปรุง Security Group
# - ปิด port 3000, 5000 (ใช้ Nginx reverse proxy)
# - จำกัด SSH เฉพาะ IP ของคุณ

# 2. ใช้ Environment Variables
# ไม่ควร hardcode ค่าสำคัญในโค้ด

# 3. ติดตั้ง fail2ban (ป้องกัน brute force SSH)
sudo apt install fail2ban -y
sudo systemctl enable fail2ban
sudo systemctl start fail2ban

# 4. Enable automatic security updates
sudo apt install unattended-upgrades -y
sudo dpkg-reconfigure --priority=low unattended-upgrades
```

### 9.2 Performance

```bash
# 1. Enable Nginx caching
# เพิ่มใน nginx/default.conf

# 2. Use CDN สำหรับ static assets

# 3. Enable gzip compression
# Nginx มี gzip enabled by default

# 4. Monitor resources
htop
docker stats
```

### 9.3 Monitoring

ติดตั้ง monitoring tools:
```bash
# Install Node Exporter (สำหรับ Prometheus)
wget https://github.com/prometheus/node_exporter/releases/download/v1.7.0/node_exporter-1.7.0.linux-amd64.tar.gz
tar xvfz node_exporter-1.7.0.linux-amd64.tar.gz
sudo cp node_exporter-1.7.0.linux-amd64/node_exporter /usr/local/bin/
rm -rf node_exporter-1.7.0.linux-amd64*

# Create systemd service
sudo nano /etc/systemd/system/node_exporter.service
```

```ini
[Unit]
Description=Node Exporter
After=network.target

[Service]
User=ubuntu
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl start node_exporter
sudo systemctl enable node_exporter
```

---

## 📋 ส่วนที่ 10: Checklist และการส่งงาน

### 10.1 Checklist

- [ ] EC2 Instance สร้างและทำงานได้
- [ ] SSH เข้าได้ปกติ
- [ ] Docker และ Docker Compose ติดตั้งแล้ว
- [ ] Nginx ติดตั้งและกำหนดค่าแล้ว
- [ ] MERN Stack application deploy แล้ว
- [ ] เข้าถึงผ่าน Public IP ได้
- [ ] API ทำงานได้ (CRUD operations)
- [ ] MongoDB เก็บข้อมูลได้
- [ ] Security Group กำหนดถูกต้อง
- [ ] Backup script สร้างแล้ว

### 10.2 การส่งงาน

**ส่งใน Microsoft Teams:**

1. **Screenshot หน้าจอ:**
   - EC2 Dashboard แสดง Instance
   - Browser เปิด application ที่ Public IP
   - Terminal แสดง `docker compose ps`
   - Application ที่ทำงานได้ (เพิ่ม/ลบ items)

2. **เอกสารรายงาน (README.md):**
```markdown
# MERN Stack Deployment Report

## ข้อมูล EC2 Instance
- Region: ap-southeast-1
- Instance Type: t2.micro
- Public IP: x.x.x.x
- OS: Ubuntu 22.04 LTS

## Services ที่ติดตั้ง
- Docker version: x.x.x
- Docker Compose version: x.x.x
- Nginx version: x.x.x

## Application URL
http://[YOUR-IP]

## Screenshots
[แนบ screenshots]

## ปัญหาที่พบและวิธีแก้
1. ปัญหา: ...
   แก้ไข: ...

## สิ่งที่ได้เรียนรู้
- ...
```

3. **ไฟล์โค้ด:**
   - docker-compose.yml
   - Backend และ Frontend code
   - Nginx configuration

---

## 🎯 สรุปและ Key Takeaways

### สิ่งที่เราได้ทำ:
1. ✅ สร้าง AWS EC2 Instance
2. ✅ ติดตั้ง Ubuntu Server พร้อม updates
3. ✅ ติดตั้ง Docker, Docker Compose, Nginx
4. ✅ Deploy MERN Stack ด้วย Docker Compose
5. ✅ กำหนดค่า Nginx เป็น Reverse Proxy
6. ✅ ทำให้ application เข้าถึงได้ผ่าน Internet

### ความรู้สำคัญ:
- **Infrastructure as Code**: ใช้ Docker Compose define infrastructure
- **Container Orchestration**: จัดการ multiple containers
- **Reverse Proxy**: Nginx เป็นตัวกลางระหว่าง user และ services
- **Cloud Computing**: Deploy บน AWS EC2
- **CI/CD Ready**: โครงสร้างพร้อมสำหรับ automated deployment

### ขั้นตอนต่อไป:
1. เพิ่ม Domain Name (Route 53)
2. ติดตั้ง SSL Certificate (Let's Encrypt)
3. Setup CI/CD pipeline (GitHub Actions)
4. Implement monitoring (Prometheus + Grafana)
5. Setup auto-scaling
6. Implement backup strategies

---

## 📚 แหล่งข้อมูลเพิ่มเติม

### เอกสารอ้างอิง
- [AWS EC2 Documentation](https://docs.aws.amazon.com/ec2/)
- [Docker Documentation](https://docs.docker.com/)
- [Nginx Documentation](https://nginx.org/en/docs/)
- [MongoDB Documentation](https://docs.mongodb.com/)

### Tutorials
- [AWS Free Tier](https://aws.amazon.com/free/)
- [Docker Compose Best Practices](https://docs.docker.com/compose/production/)
- [Nginx Reverse Proxy Guide](https://docs.nginx.com/nginx/admin-guide/web-server/reverse-proxy/)

---

## 🆘 การขอความช่วยเหลือ

หากพบปัญหา:
1. เช็ค logs: `docker compose logs -f`
2. ดู troubleshooting section ในเอกสารนี้
3. Search error message ใน Google/Stack Overflow
4. ถามใน Microsoft Teams
5. ติดต่ออาจารย์ผู้สอน

**ขอให้สนุกกับการ Deploy! 🚀**

---

**หมายเหตุ:** คู่มือนี้สร้างขึ้นสำหรับ DTI201 Full-Stack Development Course วันที่ 31 ตุลาคม 2025
