# 01 — `HEADER`

> อ่าน [README.md](README.md) ก่อน (หลักการ + กฎ Autofill มาก่อน Lookup)

---

## 1. `HEADER` ส่งต่อตามที่ user กรอก — ห้ามแก้

> **convert ไม่แก้ `HEADER`** — user กรอกอะไรมาบนหน้าจอ ให้ส่ง `HEADER` ออกไปใน payload AB **ตามนั้นทุกช่อง** ไม่เขียนทับ ไม่คำนวณ ไม่เติมค่า
>
> หน้าที่ของ convert กับ `HEADER` มีแค่ **อ่านบางค่าไปใช้** ตอน convert `BONUSBUYS[]` และ `CONDITIONS[]` ให้ครบ (ข้อ 2)

`HEADER` มีชุดเดียวต่อ payload → ทุก BBY (รวม BBY DC, BBY online และทุก Promotion No.) อยู่ใต้ `HEADER` เดียวกันนี้ ไม่ต้องแยก payload

---

## 2. ค่าที่อ่านจาก `HEADER` ไปใช้ convert (ใช้ค่าเดียวกันทุก BBY)

| ค่า | key ใน `HEADER` | ใช้กับ | บังคับกรอกบนหน้าจอ |
|---|---|---|---|
| Promotion Name | `promotionName` | `bonusBuyHeader.description`, `conditionHeader.additional_text` (สัญญา) | ✅ `*` |
| Period | `periodFrom` / `periodTo` | วันที่ใน `description`, `settlementCalendar[].settlement_date` | ✅ `*` |
| WBS No. | `wbsNumber` | `bonusBuyHeader.wbs_number` | ❌ ว่างได้ → ส่งว่าง |
| Time | `timeFrom` / `timeTo` | `bonusBuyHeader.valid_time_from` / `valid_time_to` | ❌ ถ้าว่างใช้ `00:00:00` / `23:59:59` |

รายละเอียดการใช้แต่ละค่าอยู่ที่ [02-BBY.md](02-BBY.md) และ [03-CONDITIONS.md](03-CONDITIONS.md)

---

## 3. ค่าราย BBY — อ่านจากแถวแรกของ BBY ไม่ใช่จาก `HEADER`

ค่าพวกนี้ต่างกันได้ในแต่ละ BBY (Multiple mode) → ตอน convert BBY / สัญญา ให้อ่านจาก **แถวสินค้าแถวแรกของ BBY นั้น**

| ค่า | อ่านจาก | ใช้กับ |
|---|---|---|
| Purchasing Group | `MATERIALS[].purchasing_group` | `description` (PurGroup, BU) |
| Bonus Buy Profile | `MATERIALS[].bonus_buy_profile` | กฎราย Profile, `valid_time` ของ P010 |
| Mechanic | `MATERIALS[].mechanic` | `description`, `legacy_promotion_number` |
| Promotion No. | `MATERIALS[].number_of_promotion` | prefix ใน `description` |
| Flow Type (DC/XD/DS) | `MATERIALS[].flow_type` | แยก BBY DC |
| Promo Tag | `MATERIALS[].promo_tag` | `reference_code` ของ P010 |
| Plant | `MATERIALS[].stores` | `stores`, แยก BBY online |
| Contract Type | `CONDITIONS[].conditionHeader.contract_type` | สร้างสัญญาหรือไม่ (Z2) |
| Settlement Option | `CONDITIONS[].conditionHeader.settlement_option` | จำนวนสัญญา |

> ⚠️ ช่อง `purchasingGroup`, `bonusBuyProfile`, `vendorCode`, `rebateChargeback`, `contractType` บน `HEADER` อาจเป็น "Select Mixed …" หรือค่าที่ไม่ตรงกับแถว → **อย่าใช้เป็น input** ของ BBY / สัญญา แต่ก็ **ไม่ต้องแก้** ใน `HEADER` ส่งออกไปตามที่ user กรอก

---

## 4. ตัวอย่าง — SmartWeb #43

Input `HEADER`: Promotion Name `M Price Super Shock V.1/2026`, Period `30.09.2026`–`12.11.2026`, WBS `AP.26.8883.10.CP.01`, `purchasingGroup` = `C11` · แถวแรกของ BBY: Pur Group `C03`

- Output `HEADER` = เหมือน input ทุกช่อง (`purchasingGroup` ยังเป็น `C11`)
- `bonusBuyHeader.description` ใช้ PurGroup `C03` จากแถว ไม่ใช่ `C11` จาก `HEADER`
- `bonusBuyHeader.wbs_number` = `AP.26.8883.10.CP.01` จาก `HEADER.wbsNumber`
