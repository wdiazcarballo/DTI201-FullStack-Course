# คู่มือปฏิบัติการ Docker สำหรับ DTI 201 สัปดาห์ที่ 11
## การใช้ Docker เพื่อจัดการ Container สำหรับ Backend และ Frontend

---

## 📚 สารบัญ
1. [บทนำและการเตรียมความพร้อม](#1-บทนำและการเตรียมความพร้อม)
2. [ติดตั้ง Docker](#2-ติดตั้ง-docker)
3. [พื้นฐาน Docker](#3-พื้นฐาน-docker)
4. [สร้าง Docker Image สำหรับ Web Application](#4-สร้าง-docker-image-สำหรับ-web-application)
5. [Docker Compose สำหรับ Full-Stack](#5-docker-compose-สำหรับ-full-stack)
6. [Workshop: Containerize Your MVP](#6-workshop-containerize-your-mvp)
7. [การบ้านและการประเมินผล](#7-การบ้านและการประเมินผล)

---

## 1. บทนำและการเตรียมความพร้อม

### 1.1 ทำไมต้องใช้ Docker?

จำปัญหาเหล่านี้ได้ไหม?
- 😫 "ทำไมโค้ดรันในเครื่องผมได้ แต่เครื่องเพื่อนรันไม่ได้?"
- 😤 "Project A ใช้ Node 14, Project B ใช้ Node 18 จะทำยังไง?"
- 🤯 "npm install แล้ว error เพราะ version ไม่ตรงกัน!"

**Docker คือคำตอบของปัญหาเหล่านี้!**

### 1.2 เชื่อมโยงกับ JavaScript Modules

ในสัปดาห์ที่ผ่านมาเราเรียนเรื่อง JavaScript Module Systems:
```javascript
// CommonJS
const express = require('express');

// ES6 Modules
import express from 'express';
```

ปัญหาคือ เมื่อเรา share โค้ดกับเพื่อน:
- ต้อง `npm install` ทุกครั้ง
- อาจได้ version dependencies ต่างกัน
- Environment ไม่เหมือนกัน

### 1.3 สิ่งที่จะได้เรียนในวันนี้
- ✅ สร้าง Docker Image สำหรับ Node.js Application
- ✅ ใช้ Docker Compose จัดการ Full-Stack App
- ✅ Deploy application ด้วยคำสั่งเดียว
- ✅ แก้ปัญหา "It works on my machine"

---

## 2. ติดตั้ง Docker

### 2.1 Windows
1. ดาวน์โหลด Docker Desktop จาก https://www.docker.com/products/docker-desktop
2. ติดตั้งและ restart เครื่อง
3. เปิด PowerShell ตรวจสอบ:
```powershell
docker --version
docker compose --version
```

### 2.2 macOS
1. ดาวน์โหลด Docker Desktop for Mac
2. ติดตั้งและเปิดโปรแกรม
3. เปิด Terminal ตรวจสอบ:
```bash
docker --version
docker-compose --version
```

### 2.3 Linux (Ubuntu)
```bash
# Update package index
sudo apt-get update

# Install Docker
sudo apt-get install docker.io docker-compose

# Add user to docker group
sudo usermod -aG docker $USER

# Restart หรือ logout และ login ใหม่
```

---

## 3. พื้นฐาน Docker

### 3.1 แนวคิดหลัก

Docker ทำงานด้วย 3 องค์ประกอบหลัก:

| Component | เปรียบเหมือน | คำอธิบาย |
|-----------|--------------|----------|
| **Dockerfile** | สูตรอาหาร 📝 | ไฟล์ที่บอกวิธีสร้าง Image |
| **Docker Image** | อาหารแช่แข็ง 🧊 | Template พร้อมใช้งาน |
| **Docker Container** | อาหารพร้อมเสิร์ฟ 🍽️ | Instance ที่กำลังทำงาน |

### 3.2 Lab 1: Hello Docker

#### Step 1: ทดลอง Pull และ Run Image
```bash
# ดึง Node.js image
docker pull node:14

# ดู images ที่มี
docker images

# รัน container แบบ interactive
docker run -it node:14 node
```

ใน Node.js prompt ให้ลอง:
```javascript
console.log("Hello from Docker!");
process.version
process.exit()
```

#### Step 2: รัน Container แบบ Web Server
```bash
# สร้างโฟลเดอร์ทดสอบ
mkdir docker-test
cd docker-test

# สร้างไฟล์ server.js
echo 'const http = require("http");
const server = http.createServer((req, res) => {
  res.writeHead(200, {"Content-Type": "text/plain"});
  res.end("Hello from Docker Container!");
});
server.listen(3000, () => {
  console.log("Server running on port 3000");
});' > server.js

# รัน container พร้อม mount file
docker run -p 3000:3000 -v $(pwd):/app -w /app node:14 node server.js
```

เปิด browser ไปที่ http://localhost:3000

---

## 4. สร้าง Docker Image สำหรับ Web Application

### 4.1 เตรียม Express Application

#### Step 1: สร้าง Project Structure
```bash
mkdir fullstack-docker
cd fullstack-docker
mkdir backend
cd backend
```

#### Step 2: สร้าง package.json
```json
{
  "name": "backend-api",
  "version": "1.0.0",
  "description": "Backend API with Docker",
  "main": "server.js",
  "scripts": {
    "start": "node server.js",
    "dev": "nodemon server.js"
  },
  "dependencies": {
    "express": "^4.18.2",
    "cors": "^2.8.5"
  },
  "devDependencies": {
    "nodemon": "^3.0.1"
  }
}
```

#### Step 3: สร้าง server.js
```javascript
const express = require('express');
const cors = require('cors');
const app = express();
const PORT = process.env.PORT || 8001;

// Middleware
app.use(cors());
app.use(express.json());

// Sample data
let todos = [
  { id: 1, title: 'เรียน Docker', completed: false },
  { id: 2, title: 'สร้าง Container', completed: false }
];

// Routes
app.get('/api/health', (req, res) => {
  res.json({ 
    status: 'OK', 
    message: 'Backend is running in Docker!',
    timestamp: new Date()
  });
});

app.get('/api/todos', (req, res) => {
  res.json(todos);
});

app.post('/api/todos', (req, res) => {
  const newTodo = {
    id: todos.length + 1,
    title: req.body.title,
    completed: false
  };
  todos.push(newTodo);
  res.status(201).json(newTodo);
});

app.put('/api/todos/:id', (req, res) => {
  const id = parseInt(req.params.id);
  const todo = todos.find(t => t.id === id);
  if (todo) {
    todo.completed = !todo.completed;
    res.json(todo);
  } else {
    res.status(404).json({ error: 'Todo not found' });
  }
});

app.listen(PORT, () => {
  console.log(`Backend server running on port ${PORT}`);
});
```

### 4.2 สร้าง Dockerfile

#### Step 4: สร้าง Dockerfile
```dockerfile
# เลือก base image
# alpine = Linux เวอร์ชันเล็ก ประหยัดพื้นที่
FROM node:14-alpine

# ตั้ง working directory ใน container
WORKDIR /usr/src/app

# คัดลอก package.json และ package-lock.json (ถ้ามี)
# คัดลอกไฟล์พวกนี้ก่อนเพื่อใช้ Docker cache
COPY package*.json ./

# ติดตั้ง dependencies
# ใช้ npm ci ถ้ามี package-lock.json เพื่อความเร็ว
RUN npm install

# คัดลอก source code ทั้งหมด
COPY . .

# บอกว่า app นี้ใช้ port อะไร (เพื่อ documentation)
EXPOSE 8001

# คำสั่งที่จะรันเมื่อ container start
CMD ["npm", "start"]
```

#### Step 5: สร้าง .dockerignore
```
node_modules
npm-debug.log
.env
.git
.gitignore
README.md
.vscode
```

### 4.3 Build และ Run Docker Image

#### Step 6: Build Image
```bash
# Build image with tag
docker build -t my-backend:v1 .

# ดู build process
docker build -t my-backend:v1 . --progress=plain

# ดู images ที่สร้าง
docker images | grep my-backend
```

#### Step 7: Run Container
```bash
# รันแบบ foreground
docker run -p 8001:8001 my-backend:v1

# รันแบบ background (detached)
docker run -d -p 8001:8001 --name backend-api my-backend:v1

# ดู logs
docker logs backend-api

# ดู containers ที่กำลังรัน
docker ps
```

#### Step 8: ทดสอบ API
```bash
# Test health endpoint
curl http://localhost:8001/api/health

# Get todos
curl http://localhost:8001/api/todos

# Add new todo
curl -X POST http://localhost:8001/api/todos \
  -H "Content-Type: application/json" \
  -d '{"title":"ทดสอบ Docker API"}'

# Update todo
curl -X PUT http://localhost:8001/api/todos/1
```

---

## 5. Docker Compose สำหรับ Full-Stack

### 5.1 สร้าง Frontend Application

#### Step 1: สร้าง Frontend Structure
```bash
# กลับไป root directory
cd ..
mkdir frontend
cd frontend
```

#### Step 2: สร้าง package.json
```json
{
  "name": "frontend-app",
  "version": "1.0.0",
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview"
  },
  "dependencies": {
    "axios": "^1.5.0"
  },
  "devDependencies": {
    "vite": "^4.4.0"
  }
}
```

#### Step 3: สร้าง index.html
```html
<!DOCTYPE html>
<html lang="th">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Docker Todo App</title>
  <style>
    body {
      font-family: 'Kanit', sans-serif;
      max-width: 600px;
      margin: 50px auto;
      padding: 20px;
    }
    .todo-item {
      display: flex;
      justify-content: space-between;
      padding: 10px;
      margin: 10px 0;
      border: 1px solid #ddd;
      border-radius: 5px;
    }
    .completed {
      text-decoration: line-through;
      opacity: 0.6;
    }
    button {
      padding: 5px 15px;
      cursor: pointer;
    }
    input {
      padding: 10px;
      width: 70%;
      margin-right: 10px;
    }
  </style>
</head>
<body>
  <h1>📝 Todo App with Docker</h1>
  <div>
    <input type="text" id="todoInput" placeholder="เพิ่มรายการใหม่...">
    <button onclick="addTodo()">เพิ่ม</button>
  </div>
  <div id="todoList"></div>

  <script type="module" src="/main.js"></script>
</body>
</html>
```

#### Step 4: สร้าง main.js
```javascript
const API_URL = 'http://localhost:8001/api';

let todos = [];

// Load todos on start
async function loadTodos() {
  try {
    const response = await fetch(`${API_URL}/todos`);
    todos = await response.json();
    renderTodos();
  } catch (error) {
    console.error('Error loading todos:', error);
  }
}

// Render todos to DOM
function renderTodos() {
  const todoList = document.getElementById('todoList');
  todoList.innerHTML = todos.map(todo => `
    <div class="todo-item ${todo.completed ? 'completed' : ''}">
      <span>${todo.title}</span>
      <button onclick="toggleTodo(${todo.id})">
        ${todo.completed ? '✅' : '⭕'}
      </button>
    </div>
  `).join('');
}

// Add new todo
window.addTodo = async function() {
  const input = document.getElementById('todoInput');
  if (!input.value.trim()) return;

  try {
    const response = await fetch(`${API_URL}/todos`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ title: input.value })
    });
    
    const newTodo = await response.json();
    todos.push(newTodo);
    renderTodos();
    input.value = '';
  } catch (error) {
    console.error('Error adding todo:', error);
  }
}

// Toggle todo status
window.toggleTodo = async function(id) {
  try {
    const response = await fetch(`${API_URL}/todos/${id}`, {
      method: 'PUT'
    });
    
    const updatedTodo = await response.json();
    todos = todos.map(t => t.id === id ? updatedTodo : t);
    renderTodos();
  } catch (error) {
    console.error('Error toggling todo:', error);
  }
}

// Load todos when page loads
loadTodos();
```

#### Step 5: สร้าง Dockerfile สำหรับ Frontend
```dockerfile
# Multi-stage build for production
FROM node:14-alpine as builder
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# Production stage
FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

### 5.2 สร้าง Docker Compose

#### Step 6: สร้าง docker-compose.yml ที่ root
```yaml
version: '3.8'

services:
  # Database Service
  database:
    image: postgres:14-alpine
    container_name: todo-database
    environment:
      POSTGRES_USER: todouser
      POSTGRES_PASSWORD: todopass
      POSTGRES_DB: tododb
    volumes:
      - postgres_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U todouser"]
      interval: 10s
      timeout: 5s
      retries: 5

  # Backend Service
  backend:
    build: 
      context: ./backend
      dockerfile: Dockerfile
    container_name: todo-backend
    ports:
      - "8001:8001"
    environment:
      NODE_ENV: development
      PORT: 8001
      DB_HOST: database
      DB_PORT: 5432
      DB_USER: todouser
      DB_PASSWORD: todopass
      DB_NAME: tododb
    depends_on:
      database:
        condition: service_healthy
    volumes:
      - ./backend:/usr/src/app
      - /usr/src/app/node_modules
    command: npm run dev

  # Frontend Service
  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile.dev
    container_name: todo-frontend
    ports:
      - "3000:3000"
    environment:
      - VITE_API_URL=http://localhost:8001
    depends_on:
      - backend
    volumes:
      - ./frontend:/app
      - /app/node_modules

volumes:
  postgres_data:

networks:
  default:
    name: todo-network
```

#### Step 7: สร้าง Dockerfile.dev สำหรับ Frontend Development
```dockerfile
FROM node:14-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD ["npm", "run", "dev", "--", "--host", "0.0.0.0", "--port", "3000"]
```

### 5.3 รัน Full-Stack Application

#### Step 8: เริ่มต้นใช้งาน Docker Compose
```bash
# Build images ทั้งหมด
docker-compose build

# รัน services ทั้งหมด
docker-compose up

# รันแบบ background
docker-compose up -d

# ดู logs ของทุก services
docker-compose logs -f

# ดู logs เฉพาะ service
docker-compose logs -f backend

# ดูสถานะ containers
docker-compose ps
```

#### Step 9: ทดสอบ Application
1. Frontend: http://localhost:3000
2. Backend API: http://localhost:8001/api/health
3. Database: localhost:5432 (ใช้ pgAdmin หรือ DBeaver)

#### Step 10: คำสั่งจัดการ Docker Compose
```bash
# หยุด services ทั้งหมด
docker-compose stop

# เริ่ม services ที่หยุดไว้
docker-compose start

# Restart service เฉพาะ
docker-compose restart backend

# ลบ containers และ networks
docker-compose down

# ลบรวม volumes (ระวัง! ข้อมูลจะหาย)
docker-compose down -v

# Rebuild แบบ no-cache
docker-compose build --no-cache

# Scale service (รันหลาย instances)
docker-compose up --scale backend=3
```

---

## 6. Workshop: Containerize Your MVP

### 6.1 แบ่งกลุ่มและวางแผน

#### การเตรียมตัว (15 นาที)
1. แบ่งตามทีม Sprint ปัจจุบัน
2. Clone project จาก GitHub
3. วิเคราะห์ structure ของ project

#### User Stories
```
As a developer,
I want to run the entire application with one command,
So that I can quickly set up development environment.

As a team member,
I want consistent environment across all developers,
So that we avoid "works on my machine" problems.
```

### 6.2 ขั้นตอนการ Containerize Project

#### Phase 1: Analysis (10 นาที)
```bash
# วิเคราะห์ project structure
tree -I 'node_modules|.git' -L 2

# ตรวจสอบ dependencies
cat package.json | grep dependencies

# ดู ports ที่ใช้
grep -r "PORT\|port\|listen" --include="*.js" --include="*.env"
```

#### Phase 2: Create Dockerfiles (20 นาที)

**Frontend Dockerfile Template:**
```dockerfile
FROM node:14-alpine AS development
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE [YOUR_PORT]
CMD ["npm", "run", "dev"]

FROM node:14-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
RUN npm run build

FROM nginx:alpine AS production
COPY --from=builder /app/dist /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

**Backend Dockerfile Template:**
```dockerfile
FROM node:14-alpine AS development
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE [YOUR_PORT]
CMD ["npm", "run", "dev"]

FROM node:14-alpine AS production
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
EXPOSE [YOUR_PORT]
CMD ["npm", "start"]
```

#### Phase 3: Create docker-compose.yml (15 นาที)
```yaml
version: '3.8'

services:
  # Add your services here
  your-frontend:
    build:
      context: ./path-to-frontend
      target: development
    ports:
      - "[HOST_PORT]:[CONTAINER_PORT]"
    environment:
      - NODE_ENV=development
    volumes:
      - ./path-to-frontend:/app
      - /app/node_modules

  your-backend:
    build:
      context: ./path-to-backend
      target: development
    ports:
      - "[HOST_PORT]:[CONTAINER_PORT]"
    environment:
      - NODE_ENV=development
      - DATABASE_URL=${DATABASE_URL}
    depends_on:
      - database
    volumes:
      - ./path-to-backend:/app
      - /app/node_modules

  database:
    image: postgres:14-alpine
    environment:
      POSTGRES_USER: ${DB_USER:-devuser}
      POSTGRES_PASSWORD: ${DB_PASS:-devpass}
      POSTGRES_DB: ${DB_NAME:-devdb}
    volumes:
      - db_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"

volumes:
  db_data:
```

#### Phase 4: Environment Variables (10 นาที)

สร้าง `.env` file:
```env
# Database
DB_USER=myuser
DB_PASS=mypassword
DB_NAME=mydatabase
DB_HOST=database
DB_PORT=5432

# Backend
BACKEND_PORT=8001
NODE_ENV=development

# Frontend
FRONTEND_PORT=3000
VITE_API_URL=http://localhost:8001
```

สร้าง `.env.example`:
```env
# Copy this to .env and fill in your values
DB_USER=
DB_PASS=
DB_NAME=
# ... etc
```

### 6.3 Testing และ Debugging

#### ทดสอบ Build
```bash
# Test build แต่ละ service
docker-compose build frontend
docker-compose build backend

# ดู images ที่สร้าง
docker images | head -10
```

#### Debug Common Issues

**Issue 1: Port already in use**
```bash
# หา process ที่ใช้ port
lsof -i :3000  # macOS/Linux
netstat -ano | findstr :3000  # Windows

# เปลี่ยน port ใน docker-compose.yml
ports:
  - "3001:3000"  # เปลี่ยนไปใช้ 3001
```

**Issue 2: Node modules conflict**
```dockerfile
# เพิ่มใน .dockerignore
node_modules
npm-debug.log

# ใน docker-compose.yml ใช้ anonymous volume
volumes:
  - ./app:/app
  - /app/node_modules  # Preserve container's node_modules
```

**Issue 3: Database connection failed**
```yaml
# เพิ่ม healthcheck
healthcheck:
  test: ["CMD-SHELL", "pg_isready -U postgres"]
  interval: 10s
  timeout: 5s
  retries: 5

# ใช้ depends_on with condition
depends_on:
  database:
    condition: service_healthy
```

### 6.4 Documentation Template

สร้าง `DOCKER_SETUP.md`:

```markdown
# Docker Setup Guide

## Prerequisites
- Docker Desktop installed
- Docker Compose installed
- Port 3000, 8001, 5432 available

## Quick Start
1. Clone repository
   ```bash
   git clone [repository-url]
   cd [project-name]
   ```

2. Copy environment variables
   ```bash
   cp .env.example .env
   # Edit .env with your values
   ```

3. Build and run with Docker Compose
   ```bash
   docker-compose up --build
   ```

4. Access applications
   - Frontend: http://localhost:3000
   - Backend API: http://localhost:8001
   - Database: localhost:5432

## Services
- **frontend**: React/Vue/Angular application
- **backend**: Node.js Express API
- **database**: PostgreSQL database

## Common Commands
```bash
# Start all services
docker-compose up

# Stop all services
docker-compose down

# View logs
docker-compose logs -f [service-name]

# Rebuild after changes
docker-compose build [service-name]

# Enter container shell
docker-compose exec [service-name] sh
```

## Troubleshooting
[Add common issues and solutions]
```

---

## 7. การบ้านและการประเมินผล

### 7.1 Sprint Tasks (CLO 1, 2)

#### Task 1: Complete Docker Setup (40%)
- [ ] สร้าง Dockerfile สำหรับทุก services
- [ ] สร้าง docker-compose.yml ที่ทำงานได้
- [ ] เพิ่ม health checks สำหรับ services
- [ ] ทดสอบว่า `docker-compose up` ทำงานได้

#### Task 2: Multi-Stage Build (20%)
```dockerfile
# ใช้ multi-stage build ลดขนาด image
# Development stage
FROM node:14 AS dev
# ...

# Production build stage  
FROM node:14-alpine AS build
# ...

# Production runtime stage
FROM node:14-alpine AS production
# ...
```

### 7.2 Documentation Tasks (CLO 4)

#### Task 3: README Update (20%)
เพิ่มใน README.md:
```markdown
## 🐳 Docker Setup

### Prerequisites
- Docker version 20.10+
- Docker Compose version 2.0+

### Quick Start with Docker
```bash
# Clone and setup
git clone https://github.com/yourteam/project
cd project

# Start everything
docker-compose up

# Access at http://localhost:3000
```

### Architecture
[Add architecture diagram]

### Container Structure
```
project/
├── docker-compose.yml
├── frontend/
│   └── Dockerfile
├── backend/
│   └── Dockerfile
└── database/
    └── init.sql
```
```

#### Task 4: API Documentation (10%)
ใช้ Swagger หรือ Postman สร้าง API docs:
```yaml
# docker-compose.yml เพิ่ม
swagger:
  image: swaggerapi/swagger-ui
  ports:
    - "8080:8080"
  environment:
    - SWAGGER_JSON=/api-docs/swagger.json
  volumes:
    - ./api-docs:/api-docs
```

### 7.3 Team Collaboration (CLO 3)

#### Task 5: Sprint Retrospective (10%)
บันทึกใน `SPRINT_RETRO.md`:
```markdown
# Sprint Retrospective - Docker Week

## What Went Well
- [List successes]

## What Could Be Improved
- [List challenges]

## Action Items
- [Next steps]

## Team Contributions
| Member | Role | Tasks Completed |
|--------|------|-----------------|
| | | |
```

### 7.4 เกณฑ์การประเมิน

| เกณฑ์ | คะแนน | รายละเอียด |
|-------|-------|------------|
| **Technical Implementation** | 40% | Dockerfile, docker-compose.yml ถูกต้องและใช้งานได้ |
| **Best Practices** | 20% | Multi-stage build, .dockerignore, security |
| **Documentation** | 20% | README, API docs, setup guide ชัดเจน |
| **Team Collaboration** | 10% | Git commits, code review, sprint retro |
| **Presentation** | 10% | นำเสนอชัดเจน ตอบคำถามได้ |

### 7.5 Submission Checklist

- [ ] Push code ไปที่ GitHub
- [ ] Tag version: `git tag -a week11-docker -m "Docker implementation"`
- [ ] Update README.md
- [ ] Create DOCKER_SETUP.md
- [ ] Test `docker-compose up` บน clean environment
- [ ] Submit link ใน Microsoft Teams

---

## 📚 Resources เพิ่มเติม

### Official Documentation
- [Docker Docs](https://docs.docker.com/)
- [Docker Compose](https://docs.docker.com/compose/)
- [Dockerfile Best Practices](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)

### Video Tutorials (ภาษาไทย)
- [Docker พื้นฐาน by Kong Ruksiam](https://www.youtube.com/watch?v=example)
- [Docker Compose by CodeBangkok](https://www.youtube.com/watch?v=example)

### Useful Tools
- [Docker Hub](https://hub.docker.com/) - Image registry
- [Play with Docker](https://labs.play-with-docker.com/) - Online Docker playground
- [Dive](https://github.com/wagoodman/dive) - Tool สำหรับดู Docker image layers

### Next Week Preview
สัปดาห์หน้า (Week 12): **Performance Optimization & Load Balancing**
- AWS Elastic Load Balancer
- Container orchestration basics
- Monitoring with CloudWatch
- Auto-scaling strategies

---

## 🎯 สรุป Key Takeaways

1. **Docker แก้ปัญหา "It works on my machine"** - สร้าง environment ที่เหมือนกันทุกที่
2. **Dockerfile = สูตรสร้าง Image** - เขียนครั้งเดียว build ได้ทุกที่
3. **Docker Compose = Orchestra Conductor** - จัดการหลาย containers พร้อมกัน
4. **Volumes = Persistent Storage** - เก็บข้อมูลไว้แม้ container ถูกลบ
5. **Networks = Service Communication** - ให้ containers คุยกันได้

### ⚡ Quick Commands Reference
```bash
# ชีวิตง่ายขึ้นด้วย aliases
alias dc='docker-compose'
alias dcu='docker-compose up'
alias dcd='docker-compose down'
alias dcl='docker-compose logs -f'
alias dce='docker-compose exec'
```

---

**หมายเหตุ:** คู่มือนี้ออกแบบสำหรับ DTI 201 โดยเฉพาะ หากมีคำถามหรือติดปัญหา สามารถติดต่ออาจารย์ผู้สอนผ่าน Microsoft Teams
