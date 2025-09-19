จากสไลด์ที่ 19 การเชื่อมต่อฐานข้อมูลแบบนี้ไม่ใช่ best practice มาดูวิธีที่ดีกว่ากัน:

## ❌ ปัญหาของโค้ดในสไลด์:
```javascript
// ❌ ไม่ดี - connection string อยู่ในโค้ดตรงๆ
mongoose.connect('mongodb://localhost/CRMdb', {
    useNewUrlParser: true
});
```

## ✅ Best Practices สำหรับ Database Connection:

### 1. ใช้ Environment Variables
```javascript
// .env file
MONGODB_URI=mongodb://localhost:27017/CRMdb
NODE_ENV=development
DB_USER=admin
DB_PASS=secretpassword

// config/database.js
import dotenv from 'dotenv';
dotenv.config();

export const dbConfig = {
    uri: process.env.MONGODB_URI,
    options: {
        useNewUrlParser: true,
        useUnifiedTopology: true,
        maxPoolSize: 10,
        serverSelectionTimeoutMS: 5000,
    }
};
```

### 2. สร้าง Database Connection Module แยก
```javascript
// db/connection.js
import mongoose from 'mongoose';
import { dbConfig } from '../config/database.js';

class DatabaseConnection {
    constructor() {
        this.isConnected = false;
    }

    async connect() {
        if (this.isConnected) {
            console.log('Using existing database connection');
            return;
        }

        try {
            const db = await mongoose.connect(dbConfig.uri, dbConfig.options);
            
            this.isConnected = db.connections[0].readyState === 1;
            console.log('✅ Database connected successfully');
            
            // Set up connection events
            this.setupEventListeners();
            
            return db;
        } catch (error) {
            console.error('❌ Database connection error:', error);
            process.exit(1);
        }
    }

    setupEventListeners() {
        mongoose.connection.on('connected', () => {
            console.log('Mongoose connected to MongoDB');
        });

        mongoose.connection.on('error', (err) => {
            console.error('Mongoose connection error:', err);
        });

        mongoose.connection.on('disconnected', () => {
            console.log('Mongoose disconnected');
            this.isConnected = false;
        });

        // Graceful shutdown
        process.on('SIGINT', async () => {
            await mongoose.connection.close();
            console.log('Database connection closed through app termination');
            process.exit(0);
        });
    }

    async disconnect() {
        if (!this.isConnected) {
            return;
        }

        await mongoose.disconnect();
        this.isConnected = false;
        console.log('Database disconnected');
    }
}

// Export singleton instance
export default new DatabaseConnection();
```

### 3. Connection Pooling และ Retry Logic
```javascript
// db/connectionWithRetry.js
import mongoose from 'mongoose';

const MAX_RETRIES = 5;
const RETRY_INTERVAL = 5000; // 5 seconds

export async function connectWithRetry(retries = MAX_RETRIES) {
    try {
        await mongoose.connect(process.env.MONGODB_URI, {
            useNewUrlParser: true,
            useUnifiedTopology: true,
            maxPoolSize: 10,        // Connection pool size
            minPoolSize: 2,         // Minimum connections
            socketTimeoutMS: 45000, // Socket timeout
            family: 4               // Use IPv4
        });
        
        console.log('✅ MongoDB connected');
        return mongoose.connection;
        
    } catch (error) {
        console.error(`Connection attempt failed. Retries left: ${retries - 1}`);
        console.error(error);
        
        if (retries > 1) {
            console.log(`Retrying in ${RETRY_INTERVAL / 1000} seconds...`);
            await new Promise(resolve => setTimeout(resolve, RETRY_INTERVAL));
            return connectWithRetry(retries - 1);
        } else {
            throw new Error('Failed to connect to MongoDB after maximum retries');
        }
    }
}
```

### 4. ใช้ใน index.js แบบ Clean
```javascript
// index.js
import express from 'express';
import dbConnection from './db/connection.js';
import routes from './src/routes/crmRoutes.js';

const app = express();
const PORT = process.env.PORT || 3000;

// Initialize database connection
async function startServer() {
    try {
        // Connect to database
        await dbConnection.connect();
        
        // Set up middleware
        app.use(express.json());
        app.use(express.urlencoded({ extended: true }));
        
        // Set up routes
        routes(app);
        
        // Start server
        app.listen(PORT, () => {
            console.log(`🚀 Server running on port ${PORT}`);
        });
        
    } catch (error) {
        console.error('Failed to start server:', error);
        process.exit(1);
    }
}

// Start the application
startServer();
```

### 5. Connection Health Check
```javascript
// middleware/dbHealthCheck.js
export const checkDatabaseConnection = (req, res, next) => {
    if (mongoose.connection.readyState !== 1) {
        return res.status(503).json({
            error: 'Database connection is not available'
        });
    }
    next();
};

// ใช้กับ routes ที่ต้องการ
app.use('/api', checkDatabaseConnection, routes);
```

### 6. Modern Async/Await Pattern สำหรับ Controller
```javascript
// controllers/crmController.js
import Contact from '../models/crmModel.js';

export const addNewContact = async (req, res) => {
    try {
        const newContact = new Contact(req.body);
        const savedContact = await newContact.save();
        
        res.status(201).json({
            success: true,
            data: savedContact,
            message: 'Contact created successfully'
        });
    } catch (error) {
        res.status(400).json({
            success: false,
            error: error.message || 'Failed to create contact'
        });
    }
};

export const getContacts = async (req, res) => {
    try {
        const contacts = await Contact.find()
            .limit(parseInt(req.query.limit) || 10)
            .skip(parseInt(req.query.skip) || 0);
            
        res.json({
            success: true,
            data: contacts,
            count: contacts.length
        });
    } catch (error) {
        res.status(500).json({
            success: false,
            error: 'Failed to fetch contacts'
        });
    }
};
```

### 7. สำหรับ Production: Connection String Security
```javascript
// For production with authentication
const productionUri = `mongodb://${process.env.DB_USER}:${process.env.DB_PASS}@${process.env.DB_HOST}:${process.env.DB_PORT}/${process.env.DB_NAME}?authSource=admin`;

// หรือใช้ MongoDB Atlas
const atlasUri = `mongodb+srv://${process.env.DB_USER}:${process.env.DB_PASS}@cluster0.xxxxx.mongodb.net/${process.env.DB_NAME}?retryWrites=true&w=majority`;
```

### 8. package.json scripts
```json
{
    "scripts": {
        "start": "node index.js",
        "dev": "nodemon index.js",
        "db:seed": "node scripts/seed.js",
        "db:drop": "node scripts/drop.js"
    }
}
```

## สรุป Key Points:
1. **แยก Configuration** - ไม่ฝัง connection string ในโค้ด
2. **Error Handling** - จัดการ error ทุกจุด
3. **Connection Pooling** - ใช้ connection pool เพื่อ performance
4. **Graceful Shutdown** - ปิด connection อย่างถูกต้องเมื่อปิด app
5. **Health Checks** - ตรวจสอบสถานะ connection
6. **Retry Logic** - พยายามเชื่อมต่อใหม่เมื่อล้มเหลว
7. **Modern Patterns** - ใช้ async/await แทน callbacks

วิธีนี้จะทำให้ระบบมีความเสถียรและจัดการได้ง่ายในระยะยาวครับ
