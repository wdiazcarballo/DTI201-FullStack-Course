# คู่มือฝึกปฏิบัติ: CommonJS vs ES Modules

## 📚 เตรียมความพร้อม

### สร้างโฟลเดอร์สำหรับฝึก
```bash
mkdir module-practice
cd module-practice
mkdir commonjs-demo
mkdir esm-demo
mkdir mixed-demo
```

## 🔵 ส่วนที่ 1: CommonJS Practice

### 1.1 สร้างไฟล์ทดสอบ CommonJS

```bash
cd commonjs-demo
npm init -y
```

**ไฟล์ `math-common.js`:**
```javascript
// CommonJS: การ export แบบต่างๆ

// Method 1: exports.functionName
exports.add = (a, b) => a + b;
exports.subtract = (a, b) => a - b;

// Method 2: module.exports object
module.exports.multiply = (a, b) => a * b;

// Method 3: module.exports ทั้งหมด (จะ override exports ด้านบน!)
// module.exports = {
//     add: (a, b) => a + b,
//     subtract: (a, b) => a - b
// };
```

**ไฟล์ `user-common.js`:**
```javascript
// CommonJS: export class
class User {
    constructor(name) {
        this.name = name;
    }
    
    greet() {
        return `Hello, I'm ${this.name}`;
    }
}

// Export แบบ default-like
module.exports = User;

// หรือ export as object
// module.exports = { User };
```

**ไฟล์ `test-common.js`:**
```javascript
// CommonJS: การ import/require

// Import ทั้งหมด
const math = require('./math-common');
console.log('5 + 3 =', math.add(5, 3));
console.log('5 - 3 =', math.subtract(5, 3));

// Import แบบ destructuring
const { add, subtract } = require('./math-common');
console.log('10 + 5 =', add(10, 5));

// Import class
const User = require('./user-common');
const user1 = new User('Som');
console.log(user1.greet());
```

### 🏃 ทดสอบ CommonJS
```bash
node test-common.js
```

## 🟢 ส่วนที่ 2: ES Modules Practice

### 2.1 สร้างไฟล์ทดสอบ ES Modules

```bash
cd ../esm-demo
npm init -y
```

**แก้ไข `package.json`:**
```json
{
  "name": "esm-demo",
  "version": "1.0.0",
  "type": "module"  // 👈 เพิ่มบรรทัดนี้
}
```

**ไฟล์ `math-esm.js`:**
```javascript
// ES Modules: Named exports
export const add = (a, b) => a + b;
export const subtract = (a, b) => a - b;

// Export หลายตัวพร้อมกัน
const multiply = (a, b) => a * b;
const divide = (a, b) => a / b;

export { multiply, divide };

// Default export
const mathTools = {
    power: (a, b) => Math.pow(a, b),
    sqrt: (a) => Math.sqrt(a)
};

export default mathTools;
```

**ไฟล์ `user-esm.js`:**
```javascript
// ES Modules: export class
export class User {
    constructor(name) {
        this.name = name;
    }
    
    greet() {
        return `Hello, I'm ${this.name}`;
    }
}

// Default export
export default User;

// Named export เพิ่มเติม
export const createUser = (name) => new User(name);
```

**ไฟล์ `test-esm.js`:**
```javascript
// ES Modules: การ import

// Import named exports
import { add, subtract, multiply } from './math-esm.js';  // 👈 ต้องมี .js

// Import default export
import mathTools from './math-esm.js';

// Import ทั้งหมด
import * as mathAll from './math-esm.js';

// Import ทั้ง default และ named
import User, { createUser } from './user-esm.js';

// ทดสอบ
console.log('5 + 3 =', add(5, 3));
console.log('2^3 =', mathTools.power(2, 3));
console.log('All math:', mathAll);

const user1 = new User('Noi');
console.log(user1.greet());

const user2 = createUser('Malee');
console.log(user2.greet());
```

### 🏃 ทดสอบ ES Modules
```bash
node test-esm.js
```

## 🔴 ส่วนที่ 3: ข้อผิดพลาดที่พบบ่อย

### 3.1 สร้างไฟล์แสดงข้อผิดพลาด

```bash
cd ../mixed-demo
npm init -y
echo '{"type": "module"}' > package.json
```

**ไฟล์ `errors.js`:**
```javascript
// ❌ Error 1: ใช้ require ใน ES module
// const fs = require('fs');  // Error: require is not defined

// ✅ แก้ไข:
import fs from 'fs';

// ❌ Error 2: ลืมใส่ .js
// import { helper } from './helper';  // Error: Cannot find module

// ✅ แก้ไข:
// import { helper } from './helper.js';

// ❌ Error 3: ใช้ exports ใน ES module
// exports.myFunction = () => {};  // Error: exports is not defined

// ✅ แก้ไข:
export const myFunction = () => {};

// ❌ Error 4: Import CommonJS module ผิดวิธี
// สมมติ lodash เป็น CommonJS
// import { debounce } from 'lodash';  // อาจ error

// ✅ แก้ไข:
// import lodash from 'lodash';
// const { debounce } = lodash;
```

## 🟡 ส่วนที่ 4: การใช้งานร่วมกัน

### 4.1 Import CommonJS ใน ES Module

**ไฟล์ `old-library.cjs`:** (CommonJS)
```javascript
// CommonJS module
function oldFunction() {
    return "I'm from CommonJS!";
}

module.exports = { oldFunction };
```

**ไฟล์ `use-both.js`:** (ES Module)
```javascript
// ES Module importing CommonJS
import oldLib from './old-library.cjs';

console.log(oldLib.oldFunction());

// หรือใช้ dynamic import
const dynamicImport = async () => {
    const { oldFunction } = await import('./old-library.cjs');
    console.log(oldFunction());
};

dynamicImport();
```

## 💪 แบบฝึกหัด

### Exercise 1: แปลง CommonJS เป็น ES Module

**CommonJS Version:**
```javascript
// config.js (CommonJS)
const config = {
    port: 3000,
    database: 'mongodb://localhost/mydb'
};

const getPort = () => config.port;
const getDbUrl = () => config.database;

module.exports = {
    config,
    getPort,
    getDbUrl
};
```

**TODO: แปลงเป็น ES Module:**
```javascript
// config.js (ES Module)
// เขียนโค้ดที่นี่...
```

<details>
<summary>📝 เฉลย</summary>

```javascript
// config.js (ES Module)
export const config = {
    port: 3000,
    database: 'mongodb://localhost/mydb'
};

export const getPort = () => config.port;
export const getDbUrl = () => config.database;

// หรือ
const config = {
    port: 3000,
    database: 'mongodb://localhost/mydb'
};

const getPort = () => config.port;
const getDbUrl = () => config.database;

export { config, getPort, getDbUrl };
export default config;  // optional default export
```

</details>

### Exercise 2: Debug Import/Export

**มีโค้ดที่ error:**
```javascript
// button.js
export default Button = () => {
    return '<button>Click me</button>';
};

// app.js
import { Button } from './button';  // Error!
```

**TODO: แก้ไข error**

<details>
<summary>📝 เฉลย</summary>

```javascript
// button.js
const Button = () => {
    return '<button>Click me</button>';
};
export default Button;

// app.js
import Button from './button.js';  // ✅ import default + เพิ่ม .js
```

</details>

## 📋 Cheat Sheet

### CommonJS
```javascript
// Export
exports.name = value;
module.exports = value;
module.exports = { name: value };

// Import
const module = require('./module');
const { name } = require('./module');
```

### ES Modules
```javascript
// Export
export const name = value;
export { name1, name2 };
export default value;

// Import
import name from './module.js';
import { name } from './module.js';
import * as module from './module.js';
```

## 🎯 สรุปเคล็ดลับ

1. **ตรวจสอบ package.json** - ดูว่ามี `"type": "module"` หรือไม่
2. **ใส่นามสกุล .js เสมอ** - สำหรับ ES Modules
3. **ใช้ .cjs สำหรับ CommonJS** - ถ้าอยู่ในโปรเจค ES Module
4. **อย่าผสม syntax** - เลือกใช้อย่างใดอย่างหนึ่ง
5. **Debug ทีละขั้น** - import/export ทีละไฟล์

## 🔧 Tools ช่วยแปลง

```bash
# ติดตั้ง tool ช่วยแปลง
npm install -g cjs-to-esm

# แปลงไฟล์
cjs-to-esm old-file.js > new-file.js
```
