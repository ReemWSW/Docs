# API Integration Guide for External Systems
## DOT ITG Tourist Attraction Integration

### Overview
เอกสารนี้สำหรับนักพัฒนาระบบภายนอกที่ต้องการเชื่อมต่อกับระบบ ITG เพื่อรับข้อมูลสถานประกอบการท่องเที่ยว

## Workflow การทำงานของระบบ

```mermaid
sequenceDiagram
    participant ITG as ITG System
    participant ES as External System
    participant DB as External DB
    
    Note over ITG: สถานะเปลี่ยนเป็น "2" (เผยแพร่)
    ITG->>ES: POST notification with ID
    ES->>ITG: GET /api/place/:id (ดึงข้อมูล)
    ITG-->>ES: Business Data Response
    ES->>DB: บันทึกข้อมูลลง External DB
    Note over ES: ได้ place_id ใหม่
    ES->>ITG: POST /api/place (ส่ง ID mapping)
    ITG-->>ES: Success Response
```

## Authentication

### Basic Authentication + API Key
ทุก API calls ต้องใช้ Basic Authentication และ API Key:

```http
Authorization: Basic <base64(username:password)>
apiKey: <your-api-key>
```

**Credentials:**
- Username: `{BASIC_AUTH_USERNAME}` (ตั้งค่าใน environment variable)
- Password: `{BASIC_AUTH_PASSWORD}` (ตั้งค่าใน environment variable)
- API Key: `{API_KEY}` (จะได้รับจากทีมงาน ITG)

## API Endpoints

### 1. รับ Notification (ระบบภายนอกต้องเตรียม)

ITG จะส่ง POST request มายัง E_SERVICE_URL ที่กำหนดใน environment variable

**Request:**
```http
POST {YOUR_E_SERVICE_URL}
Content-Type: application/json

{
  "id": 12345
}
```

**Response ที่ระบบภายนอกควรตอบกลับ:**
```http
HTTP/1.1 200 OK
```

### 2. ดึงข้อมูลสถานประกอบการ

**Endpoint:** `GET /api/place/:id`

**Headers:**
```http
Authorization: Basic <credentials>
apiKey: <your-api-key>
Content-Type: application/json
```

**Parameters:**
- `id` (path parameter): Attraction ID ที่ได้รับจาก notification

**Example Request:**
```http
GET /api/place/12345
Authorization: Basic dXNlcm5hbWU6cGFzc3dvcmQ=
apiKey: your-api-key-here
```

**Response (200 OK):**
```json
{
  "business_owner": {
    "name": "นายสมชาย ใจดี",
    "id_card_number": "1234567890123",
    "phone": "0812345678",
    "email": "somchai@email.com"
  },
  "businesses": {
    "itg_id": 12345,
    "place_id": null,
    "thai_name": "ร้านอาหารถิ่นเชียงใหม่",
    "english_name": "Chiang Mai Local Restaurant",
    "description_thai": "ร้านอาหารขายของอร่อยๆ",
    "description_english": "Delicious local food restaurant",
    "location": {
      "latitude": "18.7883",
      "longitude": "98.9853",
      "address": {
        "number": "123",
        "region": "เหนือ",
        "province": "เชียงใหม่",
        "district": "เมือง",
        "sub_district": "ช้างเผือก",
        "postal_code": "50300"
      }
    },
    "contact": {
      "phone": "053123456",
      "mobile": "0891234567",
      "email": "restaurant@example.com",
      "facebook": "ChiangMaiRestaurant",
      "website": "https://restaurant.com"
    },
    "operating_hours": [
      {
        "day": "จันทร์",
        "time": "08:00 - 20:00"
      },
      {
        "day": "อังคาร",
        "time": "08:00 - 20:00"
      }
    ],
    "facilities": [
      "ที่จอดรถ",
      "Wi-Fi",
      "ห้องน้ำ"
    ],
    "food_price_range": "100-300 บาท",
    "seating_capacity": "50 ที่นั่ง",
    "images": [
      "https://example.com/image1.jpg",
      "https://example.com/image2.jpg"
    ],
    "tourism_type": "ร้านอาหาร",
    "standards": [
      {
        "standards_main_received": "มาตรฐานอาหารปลอดภัย",
        "standards_sub_received": "GMP",
        "certification_start_date": "2024-01-01T00:00:00Z",
        "certification_expiry_date": "2025-01-01T00:00:00Z"
      }
    ]
  }
}
```

**Error Responses:**

```json
// 400 Bad Request
{
  "error": "Place ID is required"
}

// 400 Bad Request (Invalid ID)
{
  "error": "invalid attractionId"
}

// 500 Internal Server Error
{
  "error": "Database error message"
}
```

### 3. ส่ง ID Mapping กลับ

หลังจากบันทึกข้อมูลลงระบบแล้ว ต้องส่ง mapping ระหว่าง ITG ID และ Place ID ที่เก็บในระบบภายนอก

**Endpoint:** `POST /api/place`

**Headers:**
```http
Authorization: Basic <credentials>
apiKey: <your-api-key>
Content-Type: application/json
```

**Request Body:**
```json
{
  "itg_id": 12345,
  "place_id": "EXT_12345_ABC"
}
```

**Response (200 OK):**
```json
{
  "message": "Place ID updated successfully",
  "itg_id": 12345,
  "place_id": "EXT_12345_ABC"
}
```

**Error Responses:**

```json
// 400 Bad Request
{
  "error": "Invalid request format",
  "details": "validation error details"
}

// 500 Internal Server Error
{
  "error": "Database error message"
}
```

## Data Structure Details

### BusinessOwner Object
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| name | string | No | ชื่อเจ้าของ |
| id_card_number | string | No | เลขบัตรประชาชน/เลขทะเบียน |
| phone | string | No | หมายเลขโทรศัพท์ |
| email | string | No | อีเมล |

### BusinessEntry Object
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| itg_id | number | Yes | ID ในระบบ ITG |
| place_id | string | No | ID ในระบบภายนอก (null ในครั้งแรก) |
| thai_name | string | Yes | ชื่อภาษาไทย |
| english_name | string | No | ชื่อภาษาอังกฤษ |
| description_thai | string | No | คำอธิบายภาษาไทย |
| description_english | string | No | คำอธิบายภาษาอังกฤษ |
| location | object | Yes | ข้อมูลที่ตั้ง |
| contact | object | No | ข้อมูลติดต่อ |
| operating_hours | array | No | เวลาทำการ |
| facilities | array | No | สิ่งอำนวยความสะดวก |
| food_price_range | string | No | ช่วงราคาอาหาร |
| seating_capacity | string | No | จำนวนที่นั่ง |
| images | array | No | รูปภาพ |
| tourism_type | string | No | ประเภทการท่องเที่ยว |
| standards | array | No | มาตรฐานที่ได้รับ |

### Location Object
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| latitude | string | Yes | ละติจูด |
| longitude | string | Yes | ลองติจูด |
| address | object | Yes | ที่อยู่ |

### Address Object
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| number | string | No | เลขที่ |
| region | string | No | ภาค |
| province | string | Yes | จังหวัด |
| district | string | Yes | อำเภอ |
| sub_district | string | Yes | ตำบล |
| postal_code | string | No | รหัสไปรษณีย์ |

## Implementation Steps

### 1. เตรียม Endpoint สำหรับรับ Notification

```javascript
// Node.js Express example
app.post('/your-notification-endpoint', (req, res) => {
  const { id } = req.body;
  
  // ประมวลผล id ที่ได้รับ
  console.log(`Received attraction ID: ${id}`);
  
  // เรียกฟังก์ชันดึงข้อมูลและบันทึก
  processAttractionData(id);
  
  res.status(200).send('OK');
});
```

### 2. ดึงข้อมูลจาก ITG API

```javascript
async function fetchAttractionData(attractionId) {
  const credentials = Buffer.from(`${username}:${password}`).toString('base64');
  
  try {
    const response = await fetch(`${ITG_BASE_URL}/api/place/${attractionId}`, {
      headers: {
        'Authorization': `Basic ${credentials}`,
        'apiKey': process.env.API_KEY,
        'Content-Type': 'application/json'
      }
    });
    
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    
    return await response.json();
  } catch (error) {
    console.error('Error fetching attraction data:', error);
    throw error;
  }
}
```

### 3. บันทึกข้อมูลและส่ง ID Mapping

```javascript
async function processAttractionData(attractionId) {
  try {
    // 1. ดึงข้อมูลจาก ITG
    const attractionData = await fetchAttractionData(attractionId);
    
    // 2. บันทึกลงฐานข้อมูลของเรา
    const savedPlace = await saveToDatabase(attractionData);
    
    // 3. ส่ง ID mapping กลับ
    await sendIdMapping(attractionId, savedPlace.id);
    
  } catch (error) {
    console.error('Error processing attraction:', error);
  }
}

async function sendIdMapping(itgId, placeId) {
  const credentials = Buffer.from(`${username}:${password}`).toString('base64');
  
  const response = await fetch(`${ITG_BASE_URL}/api/place`, {
    method: 'POST',
    headers: {
      'Authorization': `Basic ${credentials}`,
      'apiKey': process.env.API_KEY,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      itg_id: itgId,
      place_id: placeId
    })
  });
  
  if (!response.ok) {
    throw new Error(`Failed to send ID mapping: ${response.status}`);
  }
  
  return await response.json();
}
```

## Environment Variables

ตั้งค่า environment variables ต่อไปนี้:

```bash
# ITG API Configuration
ITG_BASE_URL=http://itg-api.dot.go.th/api
BASIC_AUTH_USERNAME=your_username
BASIC_AUTH_PASSWORD=your_password
API_KEY=your-api-key-here

# Your notification endpoint
E_SERVICE_URL=https://your-system.com/your-notification-endpoint
```

## Error Handling

### Common Error Scenarios:
1. **Authentication Failed (401)**
   - ตรวจสอบ Basic Auth credentials
   - ตรวจสอบ API Key ว่าถูกต้องและยังไม่หมดอายุ
   
2. **Invalid Attraction ID (400)**
   - ID ไม่ถูกต้องหรือไม่มีอยู่ในระบบ
   
3. **Network Timeout**
   - ใช้ retry mechanism
   
4. **Database Error (500)**
   - ระบบ ITG มีปัญหา ให้รอสักครู่แล้วลองใหม่




