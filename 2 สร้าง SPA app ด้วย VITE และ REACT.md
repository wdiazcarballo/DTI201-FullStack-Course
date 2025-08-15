# บทที่ 2: การพัฒนา Single Page Application (SPA) ด้วย Vite และ React

## 2.1 ความรู้เบื้องต้นเกี่ยวกับ Single Page Application (SPA)

### 2.1.1 SPA คืออะไร?

Single Page Application (SPA) คือเว็บแอปพลิเคชันที่โหลดหน้าเว็บเพียงหน้าเดียวตั้งแต่เริ่มต้น และอัปเดตเนื้อหาแบบไดนามิกผ่าน JavaScript โดยไม่ต้องรีโหลดหน้าเว็บทั้งหมด ซึ่งแตกต่างจากเว็บไซต์แบบดั้งเดิม (Multi-Page Application - MPA) ที่ต้องโหลดหน้าใหม่ทุกครั้งเมื่อผู้ใช้คลิกลิงก์หรือส่งฟอร์ม

**ข้อดีของ SPA:**
- **ประสบการณ์ผู้ใช้ที่ราบรื่น**: ไม่มีการรีโหลดหน้าทำให้การใช้งานลื่นไหล
- **ประสิทธิภาพที่ดี**: โหลดเฉพาะข้อมูลที่จำเป็น ไม่ต้องโหลด HTML, CSS, JS ซ้ำ
- **การพัฒนาที่ยืดหยุ่น**: แยก Frontend และ Backend ชัดเจน

**ข้อเสียของ SPA:**
- **SEO ที่ซับซ้อน**: เนื้อหาถูกสร้างด้วย JavaScript ทำให้ search engine อ่านได้ยาก
- **การโหลดครั้งแรกช้า**: ต้องดาวน์โหลด JavaScript bundle ขนาดใหญ่
- **การจัดการ Browser History**: ต้องจัดการ URL และ navigation เอง

### 2.1.2 ทำไมต้องใช้ React สำหรับ SPA?

React เป็น JavaScript library ที่ถูกออกแบบมาเพื่อสร้าง user interface แบบ component-based ซึ่งเหมาะสมอย่างยิ่งสำหรับการพัฒนา SPA เพราะ:

1. **Declarative Approach**: เขียนโค้ดแบบบอกว่า "อยากได้อะไร" แทนที่จะบอกว่า "ทำอย่างไร"
2. **Virtual DOM**: จัดการการอัปเดต DOM อย่างมีประสิทธิภาพ
3. **Component Reusability**: สร้าง UI จาก component ที่นำกลับมาใช้ใหม่ได้
4. **Large Ecosystem**: มี library และ tools มากมายรองรับ

## 2.2 การติดตั้งและสร้างโปรเจกต์ด้วย Vite

### 2.2.1 ทำความรู้จักกับ Vite

Vite เป็น build tool ที่ถูกพัฒนาขึ้นเพื่อแก้ปัญหาความช้าของ development server ในโปรเจกต์ขนาดใหญ่ โดยมีคุณสมบัติเด่น:

- **Lightning Fast HMR**: Hot Module Replacement ที่รวดเร็วมาก
- **Native ES Modules**: ใช้ ES modules ของ browser โดยตรงในขณะพัฒนา
- **Optimized Build**: ใช้ Rollup สำหรับ production build
- **Framework Agnostic**: รองรับหลาย framework รวมถึง React, Vue, Svelte

### 2.2.2 การติดตั้ง Node.js และ npm

ก่อนเริ่มต้น ต้องติดตั้ง Node.js (แนะนำ version LTS ล่าสุด):

1. ดาวน์โหลด Node.js จาก https://nodejs.org/
2. ติดตั้งตาม installer
3. ตรวจสอบการติดตั้ง:

```bash
node -v
npm -v
```

### 2.2.3 สร้างโปรเจกต์ React ด้วย Vite

เปิด terminal แล้วรันคำสั่ง:

```bash
npm create vite@latest my-spa-app -- --template react
cd my-spa-app
npm install
npm run dev
```

โครงสร้างโปรเจกต์ที่ได้:

```
my-spa-app/
├── node_modules/
├── public/
│   └── vite.svg
├── src/
│   ├── assets/
│   ├── App.css
│   ├── App.jsx
│   ├── index.css
│   └── main.jsx
├── .gitignore
├── index.html
├── package.json
├── package-lock.json
└── vite.config.js
```

## 2.3 โครงสร้างและการทำงานของ React Components

### 2.3.1 Function Components และ JSX

React component คือ JavaScript function ที่ return JSX (JavaScript XML):

```jsx
// src/components/Welcome.jsx
function Welcome({ name }) {
  return (
    <div className="welcome">
      <h1>สวัสดี, {name}!</h1>
      <p>ยินดีต้อนรับสู่ React SPA</p>
    </div>
  );
}

export default Welcome;
```

### 2.3.2 การใช้ State และ Props

**State** - ข้อมูลภายใน component ที่สามารถเปลี่ยนแปลงได้:

```jsx
import { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0);
  
  return (
    <div>
      <p>คุณคลิกไปแล้ว {count} ครั้ง</p>
      <button onClick={() => setCount(count + 1)}>
        คลิกที่นี่
      </button>
    </div>
  );
}
```

**Props** - ข้อมูลที่ส่งเข้ามาจาก parent component:

```jsx
function UserCard({ user }) {
  return (
    <div className="user-card">
      <h3>{user.name}</h3>
      <p>{user.email}</p>
    </div>
  );
}

// การใช้งาน
<UserCard user={{ name: "สมชาย", email: "somchai@email.com" }} />
```

## 2.4 การจัดการ Routing ใน SPA

### 2.4.1 ติดตั้ง React Router

```bash
npm install react-router-dom
```

### 2.4.2 การตั้งค่า Routes

```jsx
// src/App.jsx
import { BrowserRouter as Router, Routes, Route, Link } from 'react-router-dom';
import Home from './pages/Home';
import About from './pages/About';
import Contact from './pages/Contact';

function App() {
  return (
    <Router>
      <nav>
        <ul>
          <li><Link to="/">หน้าแรก</Link></li>
          <li><Link to="/about">เกี่ยวกับเรา</Link></li>
          <li><Link to="/contact">ติดต่อเรา</Link></li>
        </ul>
      </nav>
      
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/about" element={<About />} />
        <Route path="/contact" element={<Contact />} />
      </Routes>
    </Router>
  );
}

export default App;
```

### 2.4.3 การสร้าง Page Components

```jsx
// src/pages/Home.jsx
function Home() {
  return (
    <div>
      <h1>หน้าแรก</h1>
      <p>ยินดีต้อนรับสู่เว็บไซต์ของเรา</p>
    </div>
  );
}

export default Home;
```

## 2.5 การเพิ่มประสิทธิภาพด้วย Vite

### 2.5.1 Lazy Loading Components

ใช้ React.lazy() สำหรับ code splitting:

```jsx
import { lazy, Suspense } from 'react';

const About = lazy(() => import('./pages/About'));

function App() {
  return (
    <Suspense fallback={<div>กำลังโหลด...</div>}>
      <Routes>
        <Route path="/about" element={<About />} />
      </Routes>
    </Suspense>
  );
}
```

### 2.5.2 การใช้ Environment Variables

สร้างไฟล์ `.env`:

```env
VITE_API_URL=https://api.example.com
VITE_APP_TITLE=My SPA App
```

ใช้ใน component:

```jsx
function App() {
  const apiUrl = import.meta.env.VITE_API_URL;
  const appTitle = import.meta.env.VITE_APP_TITLE;
  
  return <h1>{appTitle}</h1>;
}
```

### 2.5.3 Hot Module Replacement (HMR)

Vite มี HMR built-in ทำให้การพัฒนารวดเร็ว:

```jsx
// การเปลี่ยนแปลงใน component จะอัปเดตทันทีโดยไม่ต้อง refresh
function MyComponent() {
  return <div>เนื้อหาใหม่จะแสดงทันที!</div>;
}
```

## 2.6 Workshop: สร้าง Todo List SPA

มาลงมือสร้าง Todo List แบบ SPA กัน:

### 2.6.1 สร้าง Todo Component

```jsx
// src/components/TodoList.jsx
import { useState } from 'react';

function TodoList() {
  const [todos, setTodos] = useState([]);
  const [inputValue, setInputValue] = useState('');

  const addTodo = () => {
    if (inputValue.trim()) {
      setTodos([...todos, {
        id: Date.now(),
        text: inputValue,
        completed: false
      }]);
      setInputValue('');
    }
  };

  const toggleTodo = (id) => {
    setTodos(todos.map(todo =>
      todo.id === id ? { ...todo, completed: !todo.completed } : todo
    ));
  };

  const deleteTodo = (id) => {
    setTodos(todos.filter(todo => todo.id !== id));
  };

  return (
    <div className="todo-list">
      <h2>รายการสิ่งที่ต้องทำ</h2>
      
      <div className="todo-input">
        <input
          type="text"
          value={inputValue}
          onChange={(e) => setInputValue(e.target.value)}
          onKeyPress={(e) => e.key === 'Enter' && addTodo()}
          placeholder="เพิ่มรายการใหม่..."
        />
        <button onClick={addTodo}>เพิ่ม</button>
      </div>

      <ul>
        {todos.map(todo => (
          <li key={todo.id} className={todo.completed ? 'completed' : ''}>
            <input
              type="checkbox"
              checked={todo.completed}
              onChange={() => toggleTodo(todo.id)}
            />
            <span>{todo.text}</span>
            <button onClick={() => deleteTodo(todo.id)}>ลบ</button>
          </li>
        ))}
      </ul>
    </div>
  );
}

export default TodoList;
```

### 2.6.2 เพิ่ม CSS

```css
/* src/components/TodoList.css */
.todo-list {
  max-width: 500px;
  margin: 0 auto;
  padding: 20px;
}

.todo-input {
  display: flex;
  gap: 10px;
  margin-bottom: 20px;
}

.todo-input input {
  flex: 1;
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 4px;
}

.todo-input button {
  padding: 10px 20px;
  background: #007bff;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.todo-list ul {
  list-style: none;
  padding: 0;
}

.todo-list li {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 10px;
  border-bottom: 1px solid #eee;
}

.todo-list li.completed span {
  text-decoration: line-through;
  opacity: 0.6;
}
```

## 2.7 Best Practices สำหรับ SPA Development

### 2.7.1 การจัดโครงสร้างโฟลเดอร์

```
src/
├── components/      # Reusable components
├── pages/          # Page components
├── hooks/          # Custom hooks
├── utils/          # Utility functions
├── services/       # API services
├── assets/         # Images, fonts, etc.
└── styles/         # Global styles
```

### 2.7.2 การใช้ Custom Hooks

```jsx
// src/hooks/useLocalStorage.js
import { useState, useEffect } from 'react';

function useLocalStorage(key, initialValue) {
  const [value, setValue] = useState(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch (error) {
      return initialValue;
    }
  });

  useEffect(() => {
    try {
      window.localStorage.setItem(key, JSON.stringify(value));
    } catch (error) {
      console.error('Error saving to localStorage:', error);
    }
  }, [key, value]);

  return [value, setValue];
}

export default useLocalStorage;
```

### 2.7.3 Error Boundaries

```jsx
// src/components/ErrorBoundary.jsx
import { Component } from 'react';

class ErrorBoundary extends Component {
  state = { hasError: false };

  static getDerivedStateFromError(error) {
    return { hasError: true };
  }

  componentDidCatch(error, errorInfo) {
    console.error('Error caught:', error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      return <h1>เกิดข้อผิดพลาด กรุณารีเฟรชหน้าใหม่</h1>;
    }

    return this.props.children;
  }
}

export default ErrorBoundary;
```

## 2.8 การ Build และ Deploy

### 2.8.1 Build สำหรับ Production

```bash
npm run build
```

คำสั่งนี้จะสร้างโฟลเดอร์ `dist/` ที่มีไฟล์พร้อม deploy

### 2.8.2 Preview Production Build

```bash
npm run preview
```

### 2.8.3 การตั้งค่า vite.config.js

```javascript
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'

export default defineConfig({
  plugins: [react()],
  build: {
    outDir: 'dist',
    sourcemap: true,
    minify: 'terser'
  },
  server: {
    port: 3000,
    open: true
  }
})
```

## สรุป

ในบทนี้เราได้เรียนรู้การพัฒนา Single Page Application ด้วย Vite และ React ตั้งแต่พื้นฐาน:

1. **ความเข้าใจ SPA**: ข้อดี ข้อเสีย และเหตุผลที่เลือกใช้ React
2. **การติดตั้ง Vite**: สร้างโปรเจกต์ React อย่างรวดเร็ว
3. **React Components**: การใช้ JSX, State, และ Props
4. **Routing**: จัดการ navigation ด้วย React Router
5. **Performance**: Lazy loading และ HMR
6. **Best Practices**: โครงสร้างโปรเจกต์และ patterns ที่ควรใช้

ในบทต่อไป เราจะเรียนรู้การเชื่อมต่อ SPA กับ Backend API เพื่อสร้างแอปพลิเคชันแบบ Full-stack

## แบบฝึกหัด

1. สร้าง SPA ที่มี 3 หน้า: Home, Products, และ Cart
2. เพิ่มฟีเจอร์ค้นหาใน Todo List
3. ใช้ useLocalStorage hook เพื่อบันทึก todos
4. สร้าง Loading component สำหรับ Suspense
5. Deploy โปรเจกต์ไปยัง Vercel หรือ Netlify
