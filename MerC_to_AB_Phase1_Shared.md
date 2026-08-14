# Mer C → AB — Phase 1 (ของร่วมทุก Profile)

เอกสารนี้สำหรับรอบแรกของเว็บ: **ทำเฉพาะสิ่งที่ใช้ร่วมกันทุก profile**  
ยังไม่แยก `Gen_P010` / `Gen_D001` ฯลฯ

อ้างอิงรายละเอียด cell / ชื่อฟิลด์เต็ม: `MerC_to_AB_ExistsInAB_Mapping.md`

---

## ขอบเขต

| Phase 1 (ทำตอนนี้) | Phase 2 (ทีหลัง) |
|--------------------|------------------|
| Header + plants + วัตถุดิบแถวสินค้า | โครง Buy/Get ตาม profile |
| Lookup / รวมวันที่ / ลำดับราคา | prefix ชื่อแถว, Reference, Online text ราย profile |
| Skip rules พื้นฐาน | แยก strategy กลุ่ม A / B / C |

**ประเภทในตาราง**

| ชนิด | ความหมาย |
|------|----------|
| **ส่งตรง** | ค่าจากฟอร์ม/payload ไป AB เกือบไม่แปลง |
| **เขียนซ้ำ** | ค่าเดียว ไปหลายที่ใน AB |
| **คำนวณ** | ต้องรวม / lookup / ตัดแต่ง / เลือกตามกฎ ก่อนได้ค่า AB |

บนเว็บ mockup หลายช่องเป็น dropdown ชื่อเต็มแล้ว — ช่องที่เคยเป็น **คำนวณใน macro** อาจกลายเป็น **ส่งตรงจาก UI** ถ้าระบบใส่ค่าพร้อมใน payload (ดูคอลัมน์หมายเหตุ)

---

## 1) ส่งตรง

ค่าพร้อมแล้ว → ใส่ฟิลด์ AB ตามชื่อ

| แหล่ง (Mer C / ฟอร์ม) | → AB Field | หมายเหตุ |
|----------------------|------------|----------|
| Promotion Name | `promotionName` | ใน Excel เดิมมีสูตร — เว็บส่งค่าพร้อมได้เลย |
| Time | `validTimeFrom` | ไปที่แถว BBY (ไม่ใช่หัวโปรโมบนสุด) |
| To (Time) | `validTimeTo` | เหมือนด้านบน |
| WBS / WPS No. | `wbsNo` | ไปที่แถว BBY |
| Store ติ๊ก X ต่อสาขา | `plants[]` | แต่ละ `X` → รหัส เช่น `"13KA"` (รวมรายการ ไม่ส่งทีละ cell) |
| Material | `material` | หรืออ้างอิงกลุ่ม — ยังไม่จัด buy/get ใน Phase 1 |
| Sales Price / Normal | `normalPrice` (ของแถว) | Phase 2 ค่อยลง `get.normalPrice` |
| Sales Price / Promo | `promoPrice` (ของแถว) | ลำดับใช้คู่กับส่วนลด ดูข้อ 3 |
| Discount Amt | `discountAmt` (ของแถว) | |
| Discount % (P015 หรือ PLU) | `discountPct` (ของแถว) | คนละช่องต้นทาง แต่ปลายทางชนิดเดียวกัน |
| Sales Unit | `unit` (ของแถว) | |
| Settlement Option | `settlementOption` | ใช้เมื่อมี contract |
| Settlement Plant | `settlementPlant` | |
| Payment Terms | `paymentTerms` | |
| Payment Method | `paymentMethod` | |
| Condition Table | `conditionTable` | |
| Field Combination | `fieldCombination` | |
| Incl/Excl Biglot | `biglotSalesInclExcl` | |
| Compensate ฿/Qty, ฿/Set, % | `compensate.*` | เก็บแยกชื่อในเว็บ |

---

## 2) เขียนซ้ำ (ค่าเดียว → หลายที่)

| แหล่ง | → AB Field(s) | ทำไมซ้ำ |
|-------|---------------|---------|
| No. of / Bonus Buy (หรือเลขที่ระบบ gen) | `bonusBuyNo` → หลายจุดในแถว BBY | macro เขียน T, AN, BF, FG, GO ฯลฯ ตามฝั่งที่ profile ใช้ — Phase 1 เก็บ **ค่าเดียว** ใน DTO พอ |
| Sales Unit | `unit` อาจไปหลายคอลัมน์ตอน export | CE, HA, HP ใน Excel — เว็บเก็บค่าเดียว |
| Material / Material Grouping no. | อ้างอิงสินค้าบน Buy และ/หรือ Get | Phase 2 ตัดสินว่าเขียนฝั่งไหน; Phase 1 เก็บวัตถุดิบแถวไว้ที่เดียว |
| Time / WBS จาก header | ไป **ทุกแถว BBY** ของโปรนั้น | ไม่ใช่ช่องละค่าคนละแบบ — ค่า header ถูก copy ลงหลายแถว |
| Pur Group / Vendor / Profile | ขึ้น **หัวโปรโม** แต่มาจากแถว item | ค่าระดับกลุ่มโปรโม ใช้ร่วมทุกแถวในกลุ่ม |

> บนเว็บ: **อย่าเก็บ duplicate ใน state** — เก็บครั้งเดียว แล้วตอน export/map ตาม profile ค่อยกระจาย

---

## 3) คำนวณ / แปลง / Lookup

### 3.1 Header

| แหล่ง | → AB Field | กติกา |
|-------|------------|-------|
| Theme | `theme` | **Lookup** รหัสสั้น → ชื่อเต็ม (macro: ตัด 4 ตัว หา `Sub_condition` H→J). บนเว็บถ้า dropdown ส่งชื่อเต็มแล้ว = ส่งตรงได้ |
| Period DD + MM + YYYY | `periodStart` | **รวม 3 ช่อง** → วันที่; ปี >2500 ลบ 543; รูปแบบ `DD.MM.YYYY`. บนเว็บถ้าเป็นช่องวันที่เดียวแล้ว = ส่งตรง (ยังต้องเช็ค T+1 ตาม mockup) |
| To DD + MM + YYYY | `periodEnd` | เหมือน Period |
| Day All / Mon–Sun | `days.*` | **เงื่อนไข**: บาง profile ไม่บังคับวัน (ใน Excel ดู O14 = N/A); มี All → ใช้ all; ไม่มี → ใช้รายวัน. Phase 1 เก็บที่ user เลือกไว้ก่อน |
| (พิเศษ) Online + Compensate | `periodStart` | macro อาจ **เลื่อนวันเริ่ม −7** — บันทึกเป็นกฎพิเศษ ยังไม่ต้องทำทุกรอบ |

### 3.2 คู่หัวจาก item

| แหล่ง | → AB Field | กติกา |
|-------|------------|-------|
| Pur Group | `purchasingGroup` | **Lookup** ชื่อเต็ม |
| Bonus Buy Profile | `bonusBuyProfile` | **Lookup** ชื่อเต็ม — ใช้เป็น routing key ใน Phase 2 |
| Vendor | `vendorCode` | ว่าง → **`NOBP`** |
| Charge Back / Reason | `rebateChargeback` | **ตัดตัวอักษรแรก**; ถ้าไม่ใช่ Z2 และตัวอักษร ≠ `A` → เติม `0` นำหน้า |
| Charge Back / Contract Type | `contractType` | ขึ้นต้น **`Z2`** ถึงใส่; ไม่ใช่ → ว่าง (rebate ยังใส่ C12) |

### 3.3 สินค้า / ราคา / Mechanic

| แหล่ง | → AB Field / ค่ากลาง | กติกา |
|-------|----------------------|-------|
| มี / ไม่มี Material Group | `materialType` | มีกลุ่ม → `MGPNew`; ไม่มี → `MAT` (**คำนวณจากข้อมูลแถว**) |
| Mechanic | `buyQty`, `getQty` | **Lookup** ชีต Mechanic |
| Promo / Amt / % | ค่าส่วนลดที่ใช้จริง | **เลือกตามลำดับ**: มี `promoPrice` → ใช้ก่อน; ไม่งั้น `discountAmt`; ไม่งั้น `discountPct` |
| Discount % P015 vs PLU | `discountPct` | เลือกช่องต้นทางตาม profile/บาร์โค้ด — Phase 1 รับค่าที่ UI ส่งมาในช่อง % พอได้ |
| Compensate 3 ช่อง | ค่าที่ลง contract | ต้นทางคนละช่อง ปลายทาง Excel เดิม cell เดียว (HM) — เว็บเก็บแยก แล้วตอน export เลือกค่าที่มี |

### 3.4 Skip / แบ่งกลุ่ม (ไม่ใช่ฟิลด์ AB แต่ต้องคำนวณก่อน convert)

| แหล่ง | ผล | กติกา |
|-------|-----|-------|
| Number of Promotion | ขอบเขตกลุ่มแถว | แถวเดียวกัน = โปรชุดเดียวกัน |
| IM Reject | ข้ามแถว | ค่า `Reject` |
| Convert Status | ไม่แปลงซ้ำ | ค่า `Done` |
| Mechanic On Pack | ข้ามทั้งกลุ่ม | `B1G1 (On Pack)` / `B2G1 (On Pack)` |

---

## 4) สรุปเร็วสำหรับเพื่อนเขียนเว็บ

```
Phase 1 pipeline (ร่วมทุก profile)

1. อ่านฟอร์ม / payload
2. Skip ตามข้อ 3.4
3. สร้าง PromotionShared:
     - header: ส่งตรง + คำนวณ (วันที่, theme, days)
     - จาก item กลุ่ม: purGroup, profile, vendor, rebate, contractType (lookup/ตัดแต่ง)
     - plants[]
     - items[]: material, prices, unit, mechanic→qty, compensate
     - contract? ฟิลด์ settlement/payment… (ส่งตรง)
4. ยังไม่กระจายไป buy/get ตาม profile  ← Phase 2
```

### Checklist Phase 1

- [ ] Header ร่วมครบ (ชื่อ, theme, วันที่, วัน, เวลา, WBS)
- [ ] plants[] จากติ๊กสาขา
- [ ] lookup Pur Group / Profile / Theme (หรือได้ค่าเต็มจาก UI)
- [ ] Vendor ว่าง → NOBP
- [ ] Rebate / Contract Type ตามกฎตัดตัวอักษร + Z2
- [ ] ลำดับราคา Promo → Amt → %
- [ ] Mechanic → buyQty / getQty
- [ ] materialType MAT vs MGP
- [ ] Skip Reject / Done / On Pack
- [ ] **ยังไม่** implement Gen แยก P001/P010/D001…

---

## 5) Phase 2 — แยกราย profile

ดูสเปกครบ 9 ตัวใน **`MerC_to_AB_ByProfile.md`** (ไฟล์เดียว ส่งเพื่อนได้)

| กลุ่ม | Profile | งาน |
|------|---------|-----|
| A | P001, P010, P011, P015 | วางของแถวลง `get.*` |
| B | D001, F001 | `buy.*` + `get.*` แถวเดียว |
| C | D002, D003, F003 | แถว (A)/(B) |

ตอนนั้นค่อยเอา `bonusBuyNo` / `unit` / material ไป **เขียนซ้ำ** ตามฝั่งที่ profile ใช้
