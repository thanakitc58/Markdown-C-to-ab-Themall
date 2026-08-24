# Mechanic Lookup (ตารางเดียว)

ที่มา: ชีต Mechanic ใน Mer C Template  
ใช้ตอน C → AB: `MATERIALS[*].mechanic` → `buy.field9` / `get.getQuantity`

| Mechanic | Profile | Buy qty | Get qty | Qty/set | Skip BBY? | ใช้ตอน convert |
|----------|---------|--------:|--------:|--------:|:---------:|----------------|
| Price Off | P010 | — | 1 | 1 | | กลุ่ม A · ใส่แค่ `getQuantity` |
| PWP | P011 | — | 1 | 1 | | กลุ่ม A · ใส่แค่ `getQuantity` |
| Farmhouse/Cook Fee | P011 | — | 1 | 1 | | กลุ่ม A · ใส่แค่ `getQuantity` |
| Last Chance | P011 | — | 1 | 1 | | กลุ่ม A · ใส่แค่ `getQuantity` |
| Staff Discount | P015 | — | 1 | 1 | | กลุ่ม A · ใส่แค่ `getQuantity` |
| Time Sale | P001 | — | 1 | 1 | | กลุ่ม A · ใส่แค่ `getQuantity` |
| B1G1 | F001 | 1 | 1 | 2 | | กลุ่ม B · `field9` + `getQuantity` |
| B1G1 (On Pack) | F001 | 1 | 1 | 2 | **ใช่** | ไม่สร้าง Promotion / BBY |
| B2G1 | F001 | 2 | 1 | 3 | | กลุ่ม B |
| B2G1 (On Pack) | F001 | 2 | 1 | 3 | **ใช่** | ไม่สร้าง Promotion / BBY |
| B3G1 | F001 | 3 | 1 | 4 | | กลุ่ม B |
| B4G1 | F001 | 4 | 1 | 5 | | กลุ่ม B |
| B5G1 | F001 | 5 | 1 | 6 | | กลุ่ม B |
| B6G1 | F001 | 6 | 1 | 7 | | กลุ่ม B |
| 2For | D001 | 2 | 2 | 2 | | กลุ่ม B · `field9=2`, `getQuantity=2` |
| 3For | D001 | 3 | 3 | 3 | | กลุ่ม B |
| 4For | D001 | 4 | 4 | 4 | | กลุ่ม B |
| 5For | D001 | 5 | 5 | 5 | | กลุ่ม B |
| 6For | D001 | 6 | 6 | 6 | | กลุ่ม B |
| 15For | D001 | 15 | 15 | 15 | | กลุ่ม B |
| 1A+1B (A) | D002 | 1 | 1 | 2 | | กลุ่ม C (โครงคล้าย B) · ฝั่ง A |
| 1A+1B (B) | D002 | 1 | 1 | 2 | | ฝั่ง B |
| 2A+1B (A) | D002 | 2 | 2 | 3 | | ฝั่ง A |
| 2A+1B (B) | D002 | 1 | 1 | 3 | | ฝั่ง B |
| 2A+2B (A) | D002 | 2 | 2 | 4 | | ฝั่ง A |
| 2A+2B (B) | D002 | 2 | 2 | 4 | | ฝั่ง B |
| 1A Get 1B (A) | F003 | 1 | — | 2 | | กลุ่ม C · buy-only (`get: []`) |
| 1A Get 1B (B) | F003 | — | 1 | 2 | | get-only (`buy: []`) |
| 1A Get 2B (A) | F003 | 1 | — | 3 | | buy-only |
| 1A Get 2B (B) | F003 | — | 2 | 3 | | get-only |
| 2A Get 1B (A) | F003 | 2 | — | 3 | | buy-only |
| 2A Get 1B (B) | F003 | — | 1 | 3 | | get-only |
| A (Coupon)+B (A) | D003 | 1 | — | 2 | | กลุ่ม C · buy-only |
| A (Coupon)+B (B) | D003 | — | 1 | 2 | | get-only |

**อ่านตาราง**
- `—` = ไม่ใส่ qty ฝั่งนั้น
- **Skip BBY = ใช่** → เข้า skip rules ไม่สร้าง `BONUSBUYS`
- หา mechanic ไม่เจอในตาราง → fallback `buyQty=1`, `getQty=1` (ชั่วคราว)

**ตัวอย่าง D001:** mechanic `2For` → `buy.field9 = 2`, `get.getQuantity = 2`

JSON สำหรับโค้ด (ไฟล์เดียวกันในโฟลเดอร์นี้): [`mechanic-lookup.json`](mechanic-lookup.json)
