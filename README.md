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
