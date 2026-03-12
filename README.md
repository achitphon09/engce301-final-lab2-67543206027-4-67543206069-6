# engce301-final-set2

## URL ของ Service บน Production (Railway)

- **Auth Service**: [https://auth-service-production-559c.up.railway.app](https://auth-service-production-559c.up.railway.app)
- **Task Service**: [https://task-service-production-b94a.up.railway.app](https://task-service-production-b94a.up.railway.app)
- **User Service**: [https://user-service-production-bf73.up.railway.app](https://user-service-production-bf73.up.railway.app)

## Phase 5: Gateway Strategy

### วิธีที่เลือก: Option A (Frontend เรียก URL ของแต่ละ service โดยตรง)

**เหตุผลที่เลือก:**
- **ความง่าย (Simplicity)**: เป็นวิธีที่ง่ายที่สุดในการติดตั้ง เนื่องจากไม่ต้องตั้งค่า API Gateway หรือใช้ Nginx เป็น reverse proxy เพิ่มเติมบนสภาพแวดล้อม production
- **การเข้าถึงโดยตรง**: แต่ละ service สามารถเข้าถึงได้โดยตรงผ่าน URL สาธารณะที่ Railway กำหนดให้
- **ความเหมาะสม**: สำหรับขอบเขตของโปรเจกต์และสถาปัตยกรรม microservices บน Railway ในขณะนี้ วิธีนี้ตอบโจทย์ความต้องการโดยใช้ทรัพยากรน้อยที่สุด

**การเปรียบเทียบ:**
| Option | วิธี | ข้อดี | ข้อเสีย |
|---|---|---|---|
| **A** | **Frontend เรียก URL ของแต่ละ service โดยตรง** | ง่ายที่สุด ไม่ต้อง config เพิ่ม | Frontend ต้องรู้ URL ของทุก service |

---

## ☁️ สถาปัตยกรรมระบบ (Cloud Architecture)

```mermaid
graph TD
    Client["🌐 Web Browser (Frontend)"]
    
    subgraph "Railway.app Production"
        Auth["🔐 Auth Service<br/>(Railway)"]
        Task["📝 Task Service<br/>(Railway)"]
        User["👤 User Service<br/>(Railway)"]
        
        AuthDB[("💾 Auth DB<br/>(PostgreSQL)")]
        TaskDB[("💾 Task DB<br/>(PostgreSQL)")]
        UserDB[("💾 User DB<br/>(PostgreSQL)")]
    end

    Client -- "POST /api/auth/login" --> Auth
    Client -- "GET/POST /api/tasks" --> Task
    Client -- "GET /api/users/profile" --> User
    
    Auth --- AuthDB
    Task --- TaskDB
    User --- UserDB
    
    Task -. "JWT Verification" .-> Auth
```

---

## 🧪 วิธีทดสอบ (Testing Procedures)

สามารถทดสอบ API พื้นฐานผ่าน `curl` ได้ดังนี้:

### 1. เข้าสู่ระบบ (Login)
เพื่อขอรับ JWT Token (ใช้ Seed Users เช่น `alice@lab.local` / `alicepassword`)
```bash
curl -X POST https://auth-service-production-559c.up.railway.app/api/auth/login \
     -H "Content-Type: application/json" \
     -d '{"email": "alice@lab.local", "password": "alicepassword"}'
```

### 2. ดูข้อมูลโปรไฟล์ (User Profile)
*ให้เปลี่ยน `<YOUR_TOKEN>` เป็น Token ที่ได้จากข้อ 1*
```bash
curl -X GET https://user-service-production-bf73.up.railway.app/api/users/profile \
     -H "Authorization: Bearer <YOUR_TOKEN>"
```

### 3. ดูรายการงาน (Tasks List)
```bash
curl -X GET https://task-service-production-b94a.up.railway.app/api/tasks \
     -H "Authorization: Bearer <YOUR_TOKEN>"
```

---

## 🛠️ ปัญหาที่เจอระหว่างทำ + วิธีแก้

| ปัญหา | สาเหตุ | วิธีแก้ไข |
|---|---|---|
| **CORS Error** | Frontend เรียกข้าม Service (Auth/User/Task) บน Railway | ตั้งค่า Middleware `cors` ในทุก Service ให้รองรับ URL ของ Frontend |
| **JWT Invalid** | ค่า `JWT_SECRET` ในแต่ละ Service ไม่ตรงกัน | ตั้งค่า Environment Variable `JWT_SECRET` บน Railway ทุก Service ให้เป็นค่าเดียวกัน |
| **Database Connection** | ระบบหา Hostname ของ DB ไม่เจอเมื่อรันบน Local vs Cloud | ใช้ `DATABASE_URL` ที่ Railway กำหนดให้ใน Environment และคุมผ่าน `.env` |
| **Profile Not Found** | ตอน Login ครั้งแรกยังไม่มีข้อมูลในตาราง `user_profiles` | ตรวจสอบ Logic ใน User Service ให้ทำ Auto-Create หรือ Manual Seed ข้อมูล Profile เริ่มต้น |
