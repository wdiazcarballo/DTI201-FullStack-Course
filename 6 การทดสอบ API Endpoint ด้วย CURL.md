$ bash -lc python - <<'PY'
content = """# วิธีทดสอบ API CRUD ด้วย `curl`

คู่มือนี้อธิบายขั้นตอนการทดสอบ REST API สำหรับทรัพยากร `Contact` ในโปรเจกต์นี้ด้วยคำสั่ง `curl` ทีละคำสั่ง พร้อมตัวอย่างผลลัพธ์ที่คาดหวัง

## สิ่งที่ต้องเตรียม
- เซิร์ฟเวอร์ Express ของโปรเจกต์นี้ต้องกำลังรันอยู่ (ค่าเริ่มต้น: `npm start` จะเปิดที่ `http://localhost:3000`)
- มีฐานข้อมูล MongoDB ที่ `mongodb://localhost:27017/crmDB`
- ติดตั้ง `curl` แล้วบนเครื่อง

หากต้องการกำหนดตัวแปรพื้นฐานสำหรับคำสั่งทั้งหมด สามารถตั้งค่าแบบนี้ได้ (ไม่จำเป็น แต่ช่วยให้พิมพ์สั้นขึ้น):

```bash
BASE_URL="http://localhost:3000"
```

> หมายเหตุ: หากใช้ Windows PowerShell ให้เปลี่ยนรูปแบบตัวแปรเป็น `$BASE_URL = \"http://localhost:3000\"` และเวลานำไปใช้ต้องเขียนเป็น `$BASE_URL`

## 1. สร้างข้อมูล (Create)

ส่งคำสั่ง `POST /contact` พร้อม JSON ที่ต้องการสร้าง เช่น:

```bash
curl -X POST \"$BASE_URL/contact\" \\
  -H \"Content-Type: application/json\" \\
  -d '{\n    \"firstName\": \"Somchai\",\n    \"lastName\": \"Jaidee\",\n    \"email\": \"somchai.jaidee@example.com\",\n    \"company\": \"ACME\",\n    \"phone\": \"+66-81-234-5678\",\n    \"notes\": \"ลูกค้ากลุ่มทดลอง\"\n  }'
```

ผลลัพธ์ที่คาดหวัง: สถานะ `201 Created` และ JSON ของเอกสารที่ถูกสร้าง พร้อม `_id` ที่ MongoDB สร้างให้

> เก็บค่า `_id` ไว้สำหรับใช้ในขั้นตอนอ่าน/แก้ไข/ลบ เช่น `6650f3f7d2e96f42fb2a1234`

## 2. อ่านข้อมูลทั้งหมด (Read - Collection)

ดึงรายการ `Contact` ทั้งหมดด้วย `GET /contact`:

```bash
curl \"$BASE_URL/contact\"
```

ผลลัพธ์: สถานะ `200 OK` และอาร์เรย์ JSON ของเอกสารที่มีในฐานข้อมูล

## 3. อ่านข้อมูลตามรหัส (Read - Single Item)

แทนที่ `{id}` ด้วย `_id` ที่ได้จากขั้นตอนสร้าง:

```bash
curl \"$BASE_URL/contact/{id}\"
```

ตัวอย่าง:

```bash
curl \"$BASE_URL/contact/6650f3f7d2e96f42fb2a1234\"
```

ผลลัพธ์: สถานะ `200 OK` และ JSON ของเอกสารนั้น ถ้าไม่พบจะได้ `404 Not Found`

## 4. แก้ไขข้อมูล (Update)

ใช้ `PUT /contact/{id}` พร้อม JSON เฉพาะฟิลด์ที่ต้องการแก้ไข:

```bash
curl -X PUT \"$BASE_URL/contact/6650f3f7d2e96f42fb2a1234\" \\
  -H \"Content-Type: application/json\" \\
  -d '{\n    \"phone\": \"+66-89-999-9999\",\n    \"notes\": \"อัปเดตเบอร์โทรและบันทึก\"\n  }'
```

ผลลัพธ์: สถานะ `200 OK` และ JSON ของเอกสารหลังแก้ไข หาก ID ไม่ถูกต้องจะได้ `404 Not Found`

## 5. ลบข้อมูล (Delete)

เมื่อทดสอบจบแล้วสามารถลบเอกสารได้ด้วย `DELETE /contact/{id}`:

```bash
curl -X DELETE \"$BASE_URL/contact/6650f3f7d2e96f42fb2a1234\"
```

ผลลัพธ์: สถานะ `200 OK` พร้อมข้อความ `{ \"message\": \"CRM deleted\" }`

หาก ID ไม่พบ ระบบจะตอบ `404 Not Found`

## เคล็ดลับการทดสอบเพิ่มเติม
- หากส่ง JSON ด้วย PowerShell ให้ใช้ `"` ล้อมรอบทั้ง string แล้ว escape ด้วย `` (backtick) หรือใช้ไฟล์ `.json` แยกแล้วสั่ง `curl -d @payload.json`
- ตรวจสอบสถานะ HTTP ทุกครั้ง เพราะ Express controller จะส่งสถานะต่างกันในกรณีผิดพลาด เช่น `400 Bad Request` เมื่อข้อมูลไม่ผ่าน validation
- สามารถใช้ `jq` ต่อท้ายคำสั่ง เช่น `curl \"$BASE_URL/contact\" | jq` เพื่อจัดรูปแบบ JSON ให้อ่านง่าย (ถ้ามีการติดตั้ง `jq`)
"""
with open('CURL-CRUD-TH.md', 'w', encoding='utf-8') as f:
    f.write(content)
PY
