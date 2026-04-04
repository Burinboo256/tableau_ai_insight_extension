# 🔬 AI Insight Analyzer — Tableau Extension

วิเคราะห์ข้อมูลจาก Tableau ด้วย AI ผ่าน OpenRouter API

🌐 **Live URL:** https://burinboo256.github.io/tableau_ai_insight_extension/

---

## ✨ Features
- วิเคราะห์ข้อมูลด้วย AI (Claude, GPT-4o, Gemini, Llama)
- 3 Data Modes: KPI / Aggregate / Raw Data
- เลือก Fields สำหรับส่ง AI ได้เอง
- 2-Pass Analysis (AI เข้าใจ schema ก่อน → วิเคราะห์)
- ผลลัพธ์ 3 แท็บ: Insight / Suggestions / Follow-up
- เปรียบเทียบ 2 Worksheets พร้อมกัน
- Auto-refresh เมื่อ Tableau Filter/Mark เปลี่ยน
- Export: Copy / Markdown / HTML Report / JSON
- Anomaly Highlight ในตาราง Preview
- Word Limit + Temperature control
- ประวัติการวิเคราะห์ + ค้นหา/กรองได้

---

## 🚀 วิธีติดตั้งใน Tableau Desktop

1. ดาวน์โหลดไฟล์ `manifest.trex` จาก repo นี้
2. เปิด Tableau Desktop → Dashboard ที่ต้องการ
3. ไปที่แผง **Extensions** (ด้านซ้าย)
4. คลิก **"My Extensions"** → เลือกไฟล์ `manifest.trex`
5. คลิก **"Allow"**
6. ใส่ OpenRouter API Key ในแท็บ ⚙️ ตั้งค่า

> รับ API Key ฟรีได้ที่ [openrouter.ai/keys](https://openrouter.ai/keys)

---

## 📁 ไฟล์
| ไฟล์ | คำอธิบาย |
|---|---|
| `index.html` | Extension UI + Logic ทั้งหมด |
| `manifest.trex` | Tableau Extension manifest |

---

## ⚠️ หมายเหตุ
- API Key เก็บใน browser localStorage — สำหรับ test เท่านั้น
- ไม่ควรใช้ข้อมูล Patient จริงจนกว่าจะ deploy บน private server
