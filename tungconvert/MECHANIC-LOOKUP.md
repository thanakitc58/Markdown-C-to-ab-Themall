# Mechanic Lookup — ใช้ตอน C → AB convert

ที่มา: ชีต **Mechanic** ใน Mer C Template  
ใช้ตอนแปลง: อ่าน `MATERIALS[*].mechanic` → ได้ `buyQty` / `getQty` / โปรไฟล์ที่ผูก

## เพื่อนเอาไปใช้อย่างไร

```text
row = find Mechanic where name === materials.mechanic
buy.field9        = row.buyQty    // ถ้ามีฝั่ง buy
get.getQuantity   = row.getQty    // ถ้ามีฝั่ง get
```

- หาไม่เจอ → fallback `1` / `1` (ชั่วคราวได้ แต่ควรมีตารางครบ)
- `B1G1 (On Pack)` / `B2G1 (On Pack)` → **ข้ามทั้งกลุ่ม** (ไม่สร้าง BBY) ตาม skip rules

---

## ตารางสำหรับ convert

| Mechanic (ค่าในฟอร์ม) | Profile | Buy qty | Get qty | Qty/set | หมายเหตุตอน convert |
|------------------------|---------|--------:|--------:|--------:|---------------------|
| Price Off | P010 | — | 1 | 1 | กลุ่ม A · ใช้แค่ getQty |
| PWP | P011 | — | 1 | 1 | กลุ่ม A |
| Farmhouse/Cook Fee | P011 | — | 1 | 1 | กลุ่ม A |
| Last Chance | P011 | — | 1 | 1 | กลุ่ม A |
| Staff Discount | P015 | — | 1 | 1 | กลุ่ม A |
| Time Sale | P001 | — | 1 | 1 | กลุ่ม A |
| B1G1 | F001 | 1 | 1 | 2 | กลุ่ม B · buy+get |
| B1G1 (On Pack) | F001 | 1 | 1 | 2 | **ไม่สร้าง Promotion/BBY** · skip |
| B2G1 | F001 | 2 | 1 | 3 | |
| B2G1 (On Pack) | F001 | 2 | 1 | 3 | **ไม่สร้าง Promotion/BBY** · skip |
| B3G1 | F001 | 3 | 1 | 4 | |
| B4G1 | F001 | 4 | 1 | 5 | |
| B5G1 | F001 | 5 | 1 | 6 | |
| B6G1 | F001 | 6 | 1 | 7 | |
| **2For** | **D001** | **2** | **2** | **2** | กลุ่ม B · buy+get ก้อนเดียว |
| 3For | D001 | 3 | 3 | 3 | |
| 4For | D001 | 4 | 4 | 4 | |
| 5For | D001 | 5 | 5 | 5 | |
| 6For | D001 | 6 | 6 | 6 | |
| 15For | D001 | 15 | 15 | 15 | |
| 1A+1B (A) | D002 | 1 | 1 | 2 | กลุ่ม C โครงคล้าย B · แถวฝั่ง A |
| 1A+1B (B) | D002 | 1 | 1 | 2 | แถวฝั่ง B |
| 2A+1B (A) | D002 | 2 | 2 | 3 | |
| 2A+1B (B) | D002 | 1 | 1 | 3 | |
| 2A+2B (A) | D002 | 2 | 2 | 4 | |
| 2A+2B (B) | D002 | 2 | 2 | 4 | |
| 1A Get 1B (A) | F003 | 1 | — | 2 | กลุ่ม C · element buy-only |
| 1A Get 1B (B) | F003 | — | 1 | 2 | element get-only |
| 1A Get 2B (A) | F003 | 1 | — | 3 | buy-only |
| 1A Get 2B (B) | F003 | — | 2 | 3 | get-only |
| 2A Get 1B (A) | F003 | 2 | — | 3 | buy-only |
| 2A Get 1B (B) | F003 | — | 1 | 3 | get-only |
| A (Coupon)+B (A) | D003 | 1 | — | 2 | กลุ่ม C · buy-only |
| A (Coupon)+B (B) | D003 | — | 1 | 2 | get-only |

`—` = ฝั่งนั้นไม่ใส่ qty (หรือไม่มีฝั่งนั้นใน element)

---

## ตัวอย่างที่เพื่อนต้องจำ (D001)

ถ้า user เลือก mechanic **`2For`**:

| ใส่ใน AB | ค่าจากตาราง |
|----------|-------------|
| `bonusBuyHeader.mechanic` | `"2For"` |
| `buy[0].field9` | **2** (ไม่ใช่ 1) |
| `get[0].getQuantity` | **2** (ไม่ใช่ 1) |

เคส payload ก่อนหน้าใส่ `1` เพราะยังไม่มีตารางนี้ — พอมีตารางแล้วควรเป็น **2 / 2**

---

## JSON สำหรับใส่ในโค้ด (copy ได้)

```json
[
  { "mechanic": "Price Off", "profile": "P010", "buyQty": null, "getQty": 1 },
  { "mechanic": "PWP", "profile": "P011", "buyQty": null, "getQty": 1 },
  { "mechanic": "Farmhouse/Cook Fee", "profile": "P011", "buyQty": null, "getQty": 1 },
  { "mechanic": "Last Chance", "profile": "P011", "buyQty": null, "getQty": 1 },
  { "mechanic": "Staff Discount", "profile": "P015", "buyQty": null, "getQty": 1 },
  { "mechanic": "Time Sale", "profile": "P001", "buyQty": null, "getQty": 1 },
  { "mechanic": "B1G1", "profile": "F001", "buyQty": 1, "getQty": 1 },
  { "mechanic": "B1G1 (On Pack)", "profile": "F001", "buyQty": 1, "getQty": 1, "skip": true },
  { "mechanic": "B2G1", "profile": "F001", "buyQty": 2, "getQty": 1 },
  { "mechanic": "B2G1 (On Pack)", "profile": "F001", "buyQty": 2, "getQty": 1, "skip": true },
  { "mechanic": "B3G1", "profile": "F001", "buyQty": 3, "getQty": 1 },
  { "mechanic": "B4G1", "profile": "F001", "buyQty": 4, "getQty": 1 },
  { "mechanic": "B5G1", "profile": "F001", "buyQty": 5, "getQty": 1 },
  { "mechanic": "B6G1", "profile": "F001", "buyQty": 6, "getQty": 1 },
  { "mechanic": "2For", "profile": "D001", "buyQty": 2, "getQty": 2 },
  { "mechanic": "3For", "profile": "D001", "buyQty": 3, "getQty": 3 },
  { "mechanic": "4For", "profile": "D001", "buyQty": 4, "getQty": 4 },
  { "mechanic": "5For", "profile": "D001", "buyQty": 5, "getQty": 5 },
  { "mechanic": "6For", "profile": "D001", "buyQty": 6, "getQty": 6 },
  { "mechanic": "15For", "profile": "D001", "buyQty": 15, "getQty": 15 },
  { "mechanic": "1A+1B (A)", "profile": "D002", "buyQty": 1, "getQty": 1 },
  { "mechanic": "1A+1B (B)", "profile": "D002", "buyQty": 1, "getQty": 1 },
  { "mechanic": "2A+1B (A)", "profile": "D002", "buyQty": 2, "getQty": 2 },
  { "mechanic": "2A+1B (B)", "profile": "D002", "buyQty": 1, "getQty": 1 },
  { "mechanic": "2A+2B (A)", "profile": "D002", "buyQty": 2, "getQty": 2 },
  { "mechanic": "2A+2B (B)", "profile": "D002", "buyQty": 2, "getQty": 2 },
  { "mechanic": "1A Get 1B (A)", "profile": "F003", "buyQty": 1, "getQty": null },
  { "mechanic": "1A Get 1B (B)", "profile": "F003", "buyQty": null, "getQty": 1 },
  { "mechanic": "1A Get 2B (A)", "profile": "F003", "buyQty": 1, "getQty": null },
  { "mechanic": "1A Get 2B (B)", "profile": "F003", "buyQty": null, "getQty": 2 },
  { "mechanic": "2A Get 1B (A)", "profile": "F003", "buyQty": 2, "getQty": null },
  { "mechanic": "2A Get 1B (B)", "profile": "F003", "buyQty": null, "getQty": 1 },
  { "mechanic": "A (Coupon)+B (A)", "profile": "D003", "buyQty": 1, "getQty": null },
  { "mechanic": "A (Coupon)+B (B)", "profile": "D003", "buyQty": null, "getQty": 1 }
]
```

ไฟล์ JSON ล้วนอยู่ที่ `tungconvert/mechanic-lookup.json` (ถ้ามี) — ใช้ `import` ในโค้ดได้ตรงๆ

---

## ส่งเพื่อนยังไงดี

1. ส่งไฟล์นี้ + JSON  
2. บอกสั้นๆ: *“เลือก mechanic จากฟอร์ม → lookup ตารางนี้ได้ buy/get qty → ใส่ `field9` / `getQuantity`”*  
3. ชี้ตัวอย่าง `2For` = 2/2 (เคส D001 ที่เทสอยู่)
