# Dev handoff — Convert Mer C → AB (อ่านไฟล์นี้ก่อน)

> **ถึง:** ทีม Convert (Layout C → AB)
> **อ้างอิง:**
> - กฎทั้งหมดอ้างอิง Macro VBA เดิม: `Mer-C_Convert_To_STD.txt`
> - ชื่อ key อ้างอิง `payload-keys.ts` (`PROMOTION_PAYLOAD_KEYS`) และ payload จริง `earthcnew.json`
> - ตาราง lookup มาจาก template ของ macro `Convert_C_to_AB_Template_V2.xlsb` — ฉบับล่าสุด (ยืนยันแล้ว)

---

## ไฟล์ในชุดนี้

| ไฟล์ | เนื้อหา | ใช้ lookup |
|---|---|---|
| [01-HEADER.md](01-HEADER.md) | `HEADER` ส่งต่อตามที่ user กรอก (ไม่แก้) + ค่าที่ดึงจาก `HEADER` ไปใช้ convert | – |
| [02-BBY.md](02-BBY.md) | `BONUSBUYS[]` — `bonusBuyHeader`, `stores`, `card`, `material_or_group_type`, แยก BBY DC / online | [purchasing-group.json](lookup/purchasing-group.json) · [mechanic.json](lookup/mechanic.json) · [card-type.json](lookup/card-type.json) · [stores.json](lookup/stores.json) |
| [03-CONDITIONS.md](03-CONDITIONS.md) | `CONDITIONS[]` — สร้างสัญญาตาม macro | [plant-sales-org.json](lookup/plant-sales-org.json) · [mechanic.json](lookup/mechanic.json) |
| `lookup/*.json` | ตาราง lookup ทั้งหมด (โหลดเข้าโค้ดได้ตรงๆ) | – |

---

## หลักการ

> **convert ต้องแปลงทุกค่าตาม macro ให้เสร็จ** — หลังบ้านไม่แปลงอะไรเพิ่ม เซฟ JSON ที่ convert ส่งมาลงฐานข้อมูลตามนั้นแล้วเอาไปทำ Excel ต่อ ค่าไหนที่ convert ไม่ส่งหรือส่งผิด จะผิดไปจนถึง Excel

Mer C เป็น **Multiple mode** (เหมือน macro เดิม):

- **ค่าที่ทั้งโปรใช้ค่าเดียวกัน** → อ่านจาก `HEADER`
- **ค่าที่แต่ละ BBY ต่างกันได้** → อ่านจาก **แถวสินค้าแถวแรกของแต่ละ BBY**

> **`HEADER` ไม่ต้องแก้** — ส่งต่อตามที่ user กรอกทุกช่อง convert แค่อ่านบางค่าจาก `HEADER` ไปใช้สร้าง `BONUSBUYS[]` / `CONDITIONS[]` ให้ครบ ([01-HEADER.md](01-HEADER.md))

"แถวแรกของ BBY" = แถวแรกใน `MATERIALS[]` ที่มี `number_of_bonus_buy` เดียวกัน และ `CONDITIONS[]` ที่มี `line_number` ตรงกับแถวนั้น

---

## ⭐ กฎ Autofill มาก่อน Lookup

> ช่องไหนที่หน้าเว็บ **auto-fill มาแล้ว** (lookup จาก master หรือคำนวณ) ให้ convert **ใช้ค่านั้นตรงๆ** ไม่ต้องเปิดไฟล์ lookup หรือคำนวณซ้ำ
> ใช้ไฟล์ lookup **เฉพาะเมื่อช่องนั้นว่าง** หรือ **หน้าเว็บไม่มีช่องนั้น**

| ค่าที่ต้องใช้ | ใช้ค่า autofill จาก input ก่อน | ถ้าว่าง → lookup |
|---|---|---|
| EAN / Barcode | `MATERIALS[].barcode` | – |
| ชื่อสินค้า TH / EN | `MATERIALS[].material_description_th` / `material_description_en` | – |
| ชื่อ Vendor | `MATERIALS[].vendor_description` · `conditionHeader.vendor_name` · `businessVolumeSales[].vendor_name` | – |
| Pack Size | `MATERIALS[].pack_size` | – |
| Sales Tax (V/N) | `MATERIALS[].sales_tax` | – |
| ทุนต่อลัง (Exc.VAT) | `MATERIALS[].cost_price_case_excl_vat_normal` | – |
| ทุนต่อชิ้น (Inc.VAT) ปกติ / โปร | `MATERIALS[].cost_price_unit_inc_vat_normal` / `cost_price_unit_inc_vat_promo` | – |
| ราคาขาย ปกติ / โปร | `MATERIALS[].sales_price_normal` / `sales_price_promotion` | – |
| ฿/Qty In SAP | `MATERIALS[].compensate_quantity_in_sap` | ใช้ `compensate_baht_per_set` |
| Department | `conditionHeader.department` | [purchasing-group.json](lookup/purchasing-group.json) → `department_code` |
| Sales Organization | `conditionHeader.sales_organization` | [plant-sales-org.json](lookup/plant-sales-org.json) |
| QTY/SET | `conditionHeader.quantity` | [mechanic.json](lookup/mechanic.json) → `qty_set` |
| ค่า default ของสัญญา (`department_mer`, `start`, `end`, `set_of_field_combination`, `text_field_combination`, `selection_group`, `condition_unit`) | ค่าใน `CONDITIONS[]` | – |

ค่าที่ **ต้อง lookup เองเสมอ** (หน้าเว็บไม่มีช่อง):

| ค่า | ไฟล์ |
|---|---|
| BU ใน `description` | [purchasing-group.json](lookup/purchasing-group.json) → `bu` |
| `legacy_promotion_number` | [mechanic.json](lookup/mechanic.json) → `legacy_promotion_number` |
| `card[]` | [card-type.json](lookup/card-type.json) |
| `stores` (`KA_EXCEPT_ONLINE` / 18 สาขา) | [stores.json](lookup/stores.json) |

> ⚠️ ข้อยกเว้น: สัญญา Settlement Option `2` ที่ต้องแตกเป็นหลายสัญญาตาม Sales Org — `sales_organization` ของแต่ละสัญญาที่แตกออกมาต้องมาจาก [plant-sales-org.json](lookup/plant-sales-org.json) เพราะหน้าเว็บมีค่าเดียวต่อแถว

---

## ลำดับการยึด และขอบเขต

> **ลำดับการยึด:** ถ้าชุดเอกสารนี้ขัดกับ `profile/MerC_to_AB_*.md` ให้ **ยึดชุดนี้** (bonusBuyHeader, stores, card, `material_or_group_type`, CONDITIONS)
>
> **ขอบเขต:** ชุดนี้ยังไม่ครอบคลุมกฎ `buy[]` / `get[]` ราย Profile (ราคา, ส่วนลด, จำนวนชิ้น) — ส่วนนั้นใช้สเปกเดิมไปก่อน จะออกเอกสารเพิ่ม

---

## ข้อตกลงที่ยืนยันแล้ว

| เรื่อง | ข้อตกลง |
|---|---|
| `HEADER` | **ไม่แก้** — ส่งต่อตามที่ user กรอก convert แค่อ่านค่าไปใช้ |
| BBY DC / หลาย Promotion No. ใน payload เดียว (macro แยกไฟล์ แต่ AB มี `HEADER` ชุดเดียว) | **อยู่ใต้ `HEADER` ชุดเดียวทั้งหมด** — ไม่แยก payload |
| template `Convert_C_to_AB_Template_V2.xlsb` | **ฉบับล่าสุดแล้ว** — ไฟล์ lookup ใช้ได้เลย |

## เรื่องที่ยังเปิดอยู่

| เรื่อง | ตอนนี้ทำแบบไหน |
|---|---|
| กฎ `buy[]` / `get[]` ราย Profile | ใช้สเปกเดิมไปก่อน |
| เลข BBY ของ DC / online | ต่อเลขจาก BBY ชุดสุดท้ายใน payload |
