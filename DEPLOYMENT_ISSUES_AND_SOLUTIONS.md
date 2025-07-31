# 🚀 Deployment Issues & Solutions

## 📋 สรุปปัญหาที่พบและวิธีแก้ไข

### 1. ❌ ปัญหา: Frontend ใช้ localhost URL แทน Production URL

**อาการ:**
```
Access to fetch at 'http://localhost:5000/api/auth/login' from origin 'https://grateful-nourishment-production-68a2.up.railway.app' has been blocked by CORS policy
```

**สาเหตุ:**
- Frontend บน production ยังคงเรียก API ที่ `localhost:5000`
- ไม่มีการตั้งค่า environment variables สำหรับ production
- มีไฟล์หลายไฟล์ที่ hardcode URL

**วิธีแก้ไข:**

#### 1.1 สร้างไฟล์ Environment Variables

**Frontend/.env.production:**
```env
VITE_API_URL=https://sodeclick-production.up.railway.app
VITE_SOCKET_URL=https://sodeclick-production.up.railway.app

# App Configuration
VITE_APP_NAME=Love App
VITE_APP_VERSION=1.0.0
VITE_APP_DESCRIPTION=Elite Dating Platform

# Feature Flags
VITE_ENABLE_CHAT=true
VITE_ENABLE_PREMIUM=true
VITE_ENABLE_ADMIN=true
VITE_ENABLE_NOTIFICATIONS=true
VITE_ENABLE_ANALYTICS=false
```

**Frontend/.env.development:**
```env
VITE_API_URL=http://localhost:5000
VITE_SOCKET_URL=http://localhost:5000

# App Configuration
VITE_APP_NAME=Love App
VITE_APP_VERSION=1.0.0
VITE_APP_DESCRIPTION=Elite Dating Platform

# Feature Flags
VITE_ENABLE_CHAT=true
VITE_ENABLE_PREMIUM=true
VITE_ENABLE_ADMIN=true
VITE_ENABLE_NOTIFICATIONS=true
VITE_ENABLE_ANALYTICS=false
```

#### 1.2 แก้ไข package.json scripts

```json
{
  "scripts": {
    "dev": "vite --mode development",
    "build": "vite build --mode production",
    "build:dev": "vite build --mode development",
    "build:prod": "vite build --mode production"
  }
}
```

#### 1.3 แก้ไขไฟล์ที่ใช้ hardcode URL

**ไฟล์ที่ต้องแก้ไข:**
- `Frontend/src/components/auth/AuthModal.jsx`
- `Frontend/src/pages/auth/LoginPage.jsx`
- `Frontend/src/pages/profile/UserProfile.jsx`
- `Frontend/src/pages/admin/UserManagement.jsx`
- `Frontend/src/pages/admin/PremiumManagement.jsx`
- `Frontend/src/pages/chat/ChatPage.jsx`
- `Frontend/src/App.jsx`

**ตัวอย่างการแก้ไข:**
```javascript
// ❌ เก่า
const response = await fetch('http://localhost:5000/api/auth/login', {

// ✅ ใหม่
import { API_BASE_URL } from '../../utils/constants';
const response = await fetch(`${API_BASE_URL}/api/auth/login`, {
```

---

### 2. ❌ ปัญหา: Route `/admin` ไม่มีอยู่

**อาการ:**
```
No routes matched location "/admin"
```

**สาเหตุ:**
- ไม่มี route สำหรับ `/admin` ใน Routes.jsx
- AdminDashboard component ไม่ได้ถูก import

**วิธีแก้ไข:**

#### 2.1 เพิ่ม Route ใน Routes.jsx

```javascript
import AdminDashboard from './pages/admin/AdminDashboard';

// เพิ่ม route นี้
<Route 
  path="/admin" 
  element={
    user?.role === 'admin' ? 
    <AdminDashboard /> : 
    <div className="min-h-screen bg-black text-white flex items-center justify-center">
      <div className="text-center">
        <h1 className="text-2xl font-bold text-red-400 mb-4">ไม่มีสิทธิ์เข้าถึง</h1>
        <p className="text-zinc-400">เฉพาะ Admin เท่านั้นที่สามารถเข้าถึงหน้านี้ได้</p>
      </div>
    </div>
  } 
/>
```

---

### 3. ❌ ปัญหา: Admin User ไม่สามารถ Login ได้

**อาการ:**
```
POST https://sodeclick-production.up.railway.app/api/auth/login 401 (Unauthorized)
Login failed: อีเมล/ชื่อผู้ใช้ หรือรหัสผ่านไม่ถูกต้อง
```

**สาเหตุ:**
- Admin user ไม่มีอยู่ใน production database
- Admin user มี password เป็น null/undefined
- Password hash ไม่ถูกต้อง

**วิธีแก้ไข:**

#### 3.1 สร้าง Admin User ใหม่

```bash
cd Backend
node simpleAdminReset.js
```

#### 3.2 ตรวจสอบ Admin User

```bash
node -e "
const mongoose = require('mongoose');
const config = require('./config');
const User = require('./models/User');

mongoose.connect(config.mongodb.uri).then(async () => {
  const admin = await User.findOne({ email: 'admin@sodeclick.com' });
  console.log('Admin found:', !!admin);
  console.log('Password exists:', !!admin?.password);
  console.log('Role:', admin?.role);
  mongoose.connection.close();
});
"
```

#### 3.3 Admin Credentials

```
Username: Admin
Email: admin@sodeclick.com
Password: root77
Role: admin
```

---

### 4. ❌ ปัญหา: CORS Policy

**อาการ:**
```
Access-Control-Allow-Origin header is present on the requested resource
```

**สาเหตุ:**
- Backend CORS configuration ไม่รวม Frontend URL
- Frontend URL เปลี่ยนแปลงแต่ Backend ไม่ได้อัพเดท

**วิธีแก้ไข:**

#### 4.1 ตรวจสอบ CORS Configuration

**Backend/config.js:**
```javascript
cors: {
  origin: (() => {
    if (process.env.CORS_ORIGIN) {
      return process.env.CORS_ORIGIN.split(',').map(url => url.trim());
    }
    
    if (process.env.NODE_ENV === 'production') {
      return [
        'https://sodeclick-production.up.railway.app',
        'https://grateful-nourishment-production-68a2.up.railway.app'
      ];
    } else {
      return [
        'http://localhost:5173',
        'http://127.0.0.1:5173'
      ];
    }
  })()
}
```

---

### 5. ❌ ปัญหา: Import Path Error

**อาการ:**
```
Failed to resolve import "@/lib/utils" from "src\components\ui\button.jsx"
```

**สาเหตุ:**
- Path alias `@` ไม่ได้ตั้งค่าใน Vite config
- หรือใช้ relative path แทน

**วิธีแก้ไข:**

#### 5.1 แก้ไข Import Path

```javascript
// ❌ เก่า
import { cn } from "@/lib/utils"

// ✅ ใหม่
import { cn } from "../../lib/utils"
```

---

## 🔧 Deployment Checklist

### Pre-deployment:
- [ ] สร้างไฟล์ `.env.production` และ `.env.development`
- [ ] แก้ไข hardcode URLs ทั้งหมด
- [ ] ตรวจสอบ CORS configuration
- [ ] ตรวจสอบ import paths
- [ ] เพิ่ม routes ที่จำเป็น

### Post-deployment:
- [ ] สร้าง admin user ใน production database
- [ ] ทดสอบ API endpoints
- [ ] ทดสอบ admin login
- [ ] ตรวจสอบ console errors

---

## 🚀 Commands สำหรับแก้ไขปัญหา

### สร้าง Admin User:
```bash
cd Backend
node simpleAdminReset.js
```

### ตรวจสอบ Database Connection:
```bash
cd Backend
curl http://localhost:5000/api/health
```

### Build และ Deploy:
```bash
# Frontend
cd Frontend
npm run build
npm run preview

# หรือ
git add .
git commit -m "Fix deployment issues"
git push origin main
```

---

## 📝 หมายเหตุ

1. **Environment Variables**: Vite ต้องการ prefix `VITE_` สำหรับ client-side variables
2. **Mode Switching**: ใช้ `--mode development` หรือ `--mode production` เพื่อสลับ environment
3. **Database**: ระบบใช้ MongoDB Atlas เดียวกันทั้ง local และ production
4. **Admin Access**: ต้อง login ด้วย admin account ก่อนเข้า `/admin`

---

## 🔗 Useful Links

- **Frontend Production**: https://grateful-nourishment-production-68a2.up.railway.app
- **Backend Production**: https://sodeclick-production.up.railway.app
- **Admin Login**: https://grateful-nourishment-production-68a2.up.railway.app/admin

---

*Last updated: $(date)*