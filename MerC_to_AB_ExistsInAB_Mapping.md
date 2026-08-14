# Mer C → Mer-AB Field Mapping (exists_in_Mer-AB = Y)

ดึงจากไฟล์ `Copy of 2-1_As-Is_Structure - Mer C Promotion Template.xlsx`

- ชีต **Header**: เฉพาะแถวที่ `exists_in_Mer-AB = Y`
- ชีต **item details**: เฉพาะแถวที่ `exists_in_Mer-AB = Y`
- ปลายทาง AB = ชีต `Promotion BBY` ใน Convert C→AB (ยกเว้นที่ระบุชีตอื่น)

ใช้ส่งเพื่อนทำ convert Mer C → AB บนเว็บ

**Phase 1 (ของร่วมทุก profile — ส่งตรง / เขียนซ้ำ / คำนวณ):** ดู `MerC_to_AB_Phase1_Shared.md`

---

## 1) Header (12 ฟิลด์)

| Mer C Row | Mer C Col | Field | Input | Required | Formula | Manual | → AB Column |
|-----------|-----------|-------|-------|----------|---------|--------|-------------|
| 4 | C | Theme | Dropdown | Required | N | Y | F9 |
| 4 | M | Promotion Name | Text | - | Y | N | C8 |
| 7 | C | Period - DD | Dropdown | Required | N | Y | P11 |
| 7 | D | Period - MM | Dropdown | Required | N | Y | P11 |
| 7 | E | Period - YYYY | Dropdown | Required | N | Y | P11 |
| 7 | G | To - DD | Dropdown | Required | N | Y | P12 |
| 7 | H | To - MM | Dropdown | Required | N | Y | P12 |
| 7 | i | To - YYYY | Dropdown | Required | N | Y | P12 |
| 7 | M - T (8 cols) | Day (X) | Dropdown | Required | N | Y | P17-25 |
| 9 | C | Time | Dropdown | Optional | N | Y | W |
| 9 | F | To | Dropdown | Optional | N | Y | X |
| 9 | M | WPS No. | Text | - | Y | N | AB |

### สรุป Header ภาษาคน

| จาก Mer C | ไป AB |
|-----------|-------|
| Theme (C4) | F9 |
| Promotion Name (M4) | C8 |
| Period DD/MM/YYYY (C7–E7) | รวมเป็นวันเริ่มที่ P11 |
| To DD/MM/YYYY (G7–I7) | รวมเป็นวันจบที่ P12 |
| Day M7–T7 (All/Mon–Sun) | P17–P25 |
| Time from (C9) | W (คอลัมน์เวลาเริ่มของแถว BBY) |
| Time to (F9) | X |
| WBS / WPS No. (M9) | AB |

> หมายเหตุในต้นทาง: Promotion Name และ WBS มีสูตรใน Mer C (`has_formula=Y`) — บนเว็บอาจได้ค่าพร้อมใน payload แล้ว

### เงื่อนไขใน macro (`Fill_BBY_Header` + อ่าน header)

| Field | → AB | มีเงื่อนไข? | กติกาใน macro |
|-------|------|-------------|---------------|
| Theme | `theme` / F9 | **มี (lookup)** | ตัด `Left(Theme, 4)` → หาใน `Sub_condition` คอลัมน์ H แล้วเอาค่าคอลัมน์ J เป็นชื่อเต็ม ก่อนใส่ F9; ใน `Fill_BBY_Header` ถ้ายาว 4 ตัวจะ lookup อีกรอบเทียบคอลัมน์ J |
| Promotion Name | `promotionName` / C8 | เกือบไม่มี | ใส่ C8 ตรงๆ (ค่าจากสูตร Mer C มาแล้ว) |
| Period DD/MM/YYYY | `periodStart` / P11 | **มี** | รวม 3 ช่องเป็นวันที่; อ่านตามลำดับวันของ Excel (MDY/DMY); ถ้าปี > 2500 ลบ 543 (พ.ศ.→ค.ศ.); รูปแบบออก `DD.MM.YYYY`; บางเคส Online+Compensate **เลื่อนวันเริ่ม −7 วัน** แล้วถ้าเลยวันปัจจุบันจะใช้วันนี้แทน |
| To DD/MM/YYYY | `periodEnd` / P12 | **มี** | รวม 3 ช่อง + แปลงปี พ.ศ. เหมือน Period; รูปแบบ `DD.MM.YYYY` → P12 |
| Day (All / Mon–Sun) | `days.*` / P17, P19–P25 | **มี** | อ่าน `O14` ใน AB ก่อน: ถ้าเป็น `N/A` → **ไม่ใส่วัน**; ถ้าต้องใส่: มีค่า All (`M7`) → ใส่แค่ P17; ไม่มี All → ใส่ Mon–Sun ที่ P19–P25 |
| Time | `validTimeFrom` / W | น้อย | **ไม่ได้อยู่ใน** `Fill_BBY_Header` — ใส่ตอน `Pxxx_Header` ที่คอลัมน์ W ของแถว BBY |
| To (Time) | `validTimeTo` / X | น้อย | เหมือน Time → คอลัมน์ X ของแถว BBY |
| WPS / WBS No. | `wbsNo` / AB | น้อย | เหมือน Time — ใส่ตอน line header ที่คอลัมน์ AB ของแถว BBY (ไม่ใช่แถวหัวโปรโมบนสุด) |

**Lookup** = เอาโค้ดสั้นไปค้นตาราง master แล้วได้ค่าเต็มกลับมา (เช่น Theme `C010` → `C010 M Price Super Shock`)

#### ฟิลด์ที่ macro ใส่คู่หัว AB แต่ไม่ได้อยู่ใน 12 ช่อง Header ด้านบน

มาจาก **item details** / logic อื่น แล้วเขียนใน `Fill_BBY_Header` พร้อมกัน:

| มาจาก | → AB Field | Excel | เงื่อนไขสั้นๆ |
|-------|------------|-------|---------------|
| Pur Group (AE) | `purchasingGroup` | C9 | lookup ชื่อเต็ม (`Get_Purchasing_Group_Full_Name`) |
| Bonus Buy Profile (BA) | `bonusBuyProfile` | C10 | lookup ชื่อเต็ม (`Get_Profile_Type_Full_Name`) |
| Vendor (AJ) | `vendorCode` | L12 | ว่าง → `"NOBP"` |
| Charge Back / Reason (BQ) | `rebateChargeback` | C12 | ตัดตัวอักษรแรก; ถ้า Contract ไม่ใช่ Z2 และตัวอักษรแรก ≠ `A` จะเติม `"0"` นำหน้า |
| Charge Back / Contract Type (BR) | `contractType` | E12 | ใส่เมื่อขึ้นต้น `Z2`; ไม่ใช่ Z2 → ว่าง; C12 ใส่ rebate ทุกกรณี |

---

## 2) Item details (41 ฟิลด์)

คอลัมน์ `C To AB` = ค่าจากคอลัมน์ `column_in_Mer-AB (C To AB)` ในไฟล์ต้นฉบับ (ถ้าว่าง ใช้ `column_in_Mer-AB`)

| Mer C Col# | Letter | Field | Formula | Manual | → AB (exists) | → AB (C To AB) | Example |
|------------|--------|-------|---------|--------|---------------|----------------|---------|
| 2 | B | Store (X) / All Store | N | Y | E, Row 24 | E, Row 24 | X |
| 3 | C | Store (X) / The Mall / 13 KA (M3) | N | Y | E, Row 25 | E, Row 25 | X |
| 4 | D | Store (X) / The Mall / 14 KA (M5) | N | Y | E, Row 26 | E, Row 26 | X |
| 5 | E | Store (X) / The Mall / 15 KA (M6) | N | Y | E, Row 27 | E, Row 27 | X |
| 6 | F | Store (X) / The Mall / 16 KA (M7) | N | Y | E, Row 28 | E, Row 28 | X |
| 7 | G | Store (X) / The Mall / 17 KA (M8) | N | Y | E, Row 29 | E, Row 29 | X |
| 8 | H | Store (X) / The Mall / 33 KA (M10) | N | Y | E, Row 30 | E, Row 30 | X |
| 9 | I | Store (X) / Flagship / 30 KA (EMP) | N | Y | E, Row 31 | E, Row 31 | X |
| 10 | J | Store (X) / Flagship / 31 KA (EMQ) | N | Y | E, Row 32 | E, Row 32 | X |
| 11 | K | Store (X) / Flagship / 32 KA (EMS) | N | Y | E, Row 33 | E, Row 33 | X |
| 12 | L | Store (X) / Flagship / 34 KA (SPR) | N | Y | E, Row 34 | E, Row 34 | X |
| 13 | M | Store (X) / Stand Alone / 35 KA (BP) | - | - | E - i, Row 35 (No longer in use) | E - i, Row 35 (No longer in use) | X |
| 14 | N | Store (X) / Stand Alone / 60 KA (T21) | N | Y | E, Row 36 | E, Row 36 | X |
| 15 | O | Store (X) / Stand Alone / 61 KA (PMN) | N | Y | E, Row 37 | E, Row 37 | X |
| 17 | Q | Store (X) / Stand Alone / 65 KA (MRT LP) | N | Y | E, Row 39 | E, Row 39 | X |
| 18 | R | Store (X) / Stand Alone / 66 KA (DSV BN) | N | Y | E, Row 42 | E, Row 42 | X |
| 19 | S | Store (X) / Stand Alone / 67 KA (DSV PT) | N | Y | E, Row 40 | E, Row 40 | X |
| 28 | AB | No. of / Material Grouping | N | Y | AQ, Bi | AQ, Bi | 1.0 |
| 29 | AC | No. of / Bonus Buy | N | N | T, AN, BF, FG, GN | T, AN, BF, FG, GO | 1.0 |
| 31 | AE | Pur Group | Y | N | C9 | C9 | C01 |
| 32 | AF | Material | N | Y | E ในชีท Material Grouping เริ่มบรรทัดที่ 4 | E ในชีท Material Grouping เริ่มบรรทัดที่ 4 | 1.001032606E9 |
| 36 | AJ | Vendor | Y | N | L12 | L12 | AAB03 |
| 40 | AN | Sales Unit | Y | N | CE, HL | CE, HA, HP | EA |
| 51 | AY | Sales Price / Normal | Y | N | BM | BM | 790.0 |
| 52 | AZ | Sales Price / Promo | N | Y | BW | BW | 690.0 |
| 53 | BA | Bonus Buy Profile | Y | N | C10 | C10 | P010 |
| 55 | BC | Discount Amt | - | - | BZ | BZ |  |
| 56 | BD | Discount % / For P015 | N | Y | CA | CA |  |
| 58 | BF | Discount % / PLU | Y | N | CA | CA |  |
| 59 | BG | Compensate / ฿/Qty In SAP | Y | N | Hi | HM | 10.93 |
| 60 | BH | Compensate / ฿/Set | N | Y | Hi | HM | 32.79 |
| 61 | BI | Compensate / % | N | Y | Hi | HM |  |
| 69 | BQ | Charge Back / Reason | Y | N | C12 | C12 | A - สร้าง Contract หลังจบรายการ Promotion/Combine BBY/Fix Amount |
| 70 | BR | Charge Back / Contract Type | Y | N | E12 | E12 | Z200 - Charge Back - Sales Base w/o Vat |
| 71 | BS | Charge Back / Contract Type Z200, Z222 Incl. or Excl. ZBL - Biglot Sales | Y | N | GH | Gi | Inclusive |
| 72 | BT | Charge Back / Settlement Option | Y | N | FE | FE | 1 - Settlement to Settlement Plant |
| 73 | BU | Charge Back / Settlement Plant | Y | N | FF | FF | 13KA |
| 74 | BV | Charge Back / Payment Terms | N | Y | EX | EX | RA30 - 30 Days from invoice date |
| 75 | BW | Charge Back / Payment Method | Y | N | EY | EY | A - Check A/C Payee Only |
| 76 | BX | Charge Back / Condition Table | Y | N | GZ | HD | V 163 - เรียกเก็บเงิน By Material |
| 77 | BY | Charge Back / Field Combination | Y | N | GF & GG | GG & GH | Z212 - Plant,Material,Bonus Buy |

### จัดกลุ่ม Item ตามประเภท

#### 2.1 ร้าน / Plant (ติ๊ก X → คอลัมน์ E แถวที่ระบุใน AB)

| Mer C | → AB |
|-------|------|
| Store (X) / All Store (B) | E, Row 24 |
| Store (X) / The Mall / 13 KA (M3) (C) | E, Row 25 |
| Store (X) / The Mall / 14 KA (M5) (D) | E, Row 26 |
| Store (X) / The Mall / 15 KA (M6) (E) | E, Row 27 |
| Store (X) / The Mall / 16 KA (M7) (F) | E, Row 28 |
| Store (X) / The Mall / 17 KA (M8) (G) | E, Row 29 |
| Store (X) / The Mall / 33 KA (M10) (H) | E, Row 30 |
| Store (X) / Flagship / 30 KA (EMP) (I) | E, Row 31 |
| Store (X) / Flagship / 31 KA (EMQ) (J) | E, Row 32 |
| Store (X) / Flagship / 32 KA (EMS) (K) | E, Row 33 |
| Store (X) / Flagship / 34 KA (SPR) (L) | E, Row 34 |
| Store (X) / Stand Alone / 35 KA (BP) (M) | E - i, Row 35 (No longer in use) |
| Store (X) / Stand Alone / 60 KA (T21) (N) | E, Row 36 |
| Store (X) / Stand Alone / 61 KA (PMN) (O) | E, Row 37 |
| Store (X) / Stand Alone / 65 KA (MRT LP) (Q) | E, Row 39 |
| Store (X) / Stand Alone / 66 KA (DSV BN) (R) | E, Row 42 |
| Store (X) / Stand Alone / 67 KA (DSV PT) (S) | E, Row 40 |

#### 2.2 สินค้า / ราคา / Profile

| Mer C | Letter | → AB (C To AB) |
|-------|--------|----------------|
| No. of / Material Grouping | AB | AQ, Bi |
| No. of / Bonus Buy | AC | T, AN, BF, FG, GO |
| Pur Group | AE | C9 |
| Material | AF | E ในชีท Material Grouping เริ่มบรรทัดที่ 4 |
| Vendor | AJ | L12 |
| Sales Unit | AN | CE, HA, HP |
| Sales Price / Normal | AY | BM |
| Sales Price / Promo | AZ | BW |
| Bonus Buy Profile | BA | C10 |
| Discount Amt | BC | BZ |
| Discount % / For P015 | BD | CA |
| Discount % / PLU | BF | CA |

#### 2.3 Compensate / Charge Back / Contract

| Mer C | Letter | → AB (C To AB) |
|-------|--------|----------------|
| Compensate / ฿/Qty In SAP | BG | HM |
| Compensate / ฿/Set | BH | HM |
| Compensate / % | BI | HM |
| Charge Back / Reason | BQ | C12 |
| Charge Back / Contract Type | BR | E12 |
| Charge Back / Contract Type Z200, Z222 Incl. or Excl. ZBL - Biglot Sales | BS | Gi |
| Charge Back / Settlement Option | BT | FE |
| Charge Back / Settlement Plant | BU | FF |
| Charge Back / Payment Terms | BV | EX |
| Charge Back / Payment Method | BW | EY |
| Charge Back / Condition Table | BX | HD |
| Charge Back / Field Combination | BY | GG & GH |

---

## 3) จุดที่เพื่อนควรรู้ตอนทำเว็บ

1. **ช่องเดียวต้นทาง → หลายช่องปลายทาง** เช่น Bonus Buy no. → `T, AN, BF, FG, GO` / บนเว็บ = `bonusBuyNo`
2. **วันที่ 3 ช่องใน Mer C รวมเป็น 1 ช่องใน AB** → เว็บใช้ `periodStart` / `periodEnd`
3. **บางช่องใน Mer C เป็นสูตร** (`has_formula=Y`) เช่น Pur Group, Vendor, Sales Unit, Profile, Chargeback หลายช่อง — บนเว็บอาจคำนวณไว้ใน payload แล้ว
4. **Material (AF)** ในเอกสารนี้ระบุไปชีท Material Grouping ไม่ใช่แค่เซลล์บน Promotion BBY
5. เอกสารนี้เป็น mapping ภาพรวม C→AB — **แต่ละ Profile (เช่น D001) อาจใส่ไม่ครบทุกช่องในตาราง** ต้องดูกติกา convert เพิ่ม
6. **ชื่อฟิลด์ AB สำหรับเว็บ** ดูข้อ 5 — อย่าผูก logic เว็บกับ cell โดยตรง

---

## 4) แหล่งที่มา

| รายการ | ค่า |
|--------|-----|
| ไฟล์ต้นทาง | `Copy of 2-1_As-Is_Structure - Mer C Promotion Template.xlsx` |
| ชีต | `Header`, `item details` |
| เงื่อนไขดึง | `exists_in_Mer-AB = Y` |
| จำนวน Header | 12 |
| จำนวน Item | 41 |

---

## 5) AB Field names (สำหรับเว็บ)

ตารางด้านบนเป็น **Excel cell** สำหรับ gen ไฟล์ AB  
ตารางด้านล่างเป็น **ชื่อฟิลด์** ที่ logic / DTO บนเว็บควรใช้ — cell เก็บไว้แค่ตอน export Excel

ชื่ออ้างจาก label ใน `Promotion BBY` + mockup MerABC (ชีต MerC / Promotion Header)

### 5.1 Header

| Mer C Field | → AB Field (เว็บ) | Excel cell |
|-------------|-------------------|------------|
| Theme (C4) | `theme` | F9 |
| Promotion Name (M4) | `promotionName` | C8 |
| Period DD/MM/YYYY (C7–E7) | `periodStart` | P11 |
| To DD/MM/YYYY (G7–I7) | `periodEnd` | P12 |
| Day / All (M7) | `days.all` | P17 |
| Day / Mon–Sun (N7–T7) | `days.mon` … `days.sun` | P19–P25 |
| Time (C9) | `validTimeFrom` | W (แถว BBY) |
| To (F9) | `validTimeTo` | X (แถว BBY) |
| WPS / WBS No. (M9) | `wbsNo` | AB (แถว BBY) |

### 5.2 Plants

บนเว็บเก็บเป็นรายการรหัสสาขาที่ถูกเลือก ไม่ใช่ cell ทีละแถว

| Mer C Field | → AB Field (เว็บ) | Excel cell |
|-------------|-------------------|------------|
| Store (X) / All Store | `plants` รวม `"ALL"` หรือ flag `allStore` | E, Row 24 |
| Store (X) / 13 KA (M3) | `plants` → `"13KA"` | E, Row 25 |
| Store (X) / 14 KA (M5) | `plants` → `"14KA"` | E, Row 26 |
| Store (X) / 15 KA (M6) | `plants` → `"15KA"` | E, Row 27 |
| Store (X) / 16 KA (M7) | `plants` → `"16KA"` | E, Row 28 |
| Store (X) / 17 KA (M8) | `plants` → `"17KA"` | E, Row 29 |
| Store (X) / 33 KA (M10) | `plants` → `"33KA"` | E, Row 30 |
| Store (X) / 30 KA (EMP) | `plants` → `"30KA"` | E, Row 31 |
| Store (X) / 31 KA (EMQ) | `plants` → `"31KA"` | E, Row 32 |
| Store (X) / 32 KA (EMS) | `plants` → `"32KA"` | E, Row 33 |
| Store (X) / 34 KA (SPR) | `plants` → `"34KA"` | E, Row 34 |
| Store (X) / 35 KA (BP) | *(ไม่ใช้แล้ว)* | E–I, Row 35 |
| Store (X) / 60 KA (T21) | `plants` → `"60KA"` | E, Row 36 |
| Store (X) / 61 KA (PMN) | `plants` → `"61KA"` | E, Row 37 |
| Store (X) / 65 KA (MRT LP) | `plants` → `"65KA"` | E, Row 39 |
| Store (X) / 66 KA (DSV BN) | `plants` → `"66KA"` | E, Row 42 |
| Store (X) / 67 KA (DSV PT) | `plants` → `"67KA"` | E, Row 40 |

### 5.3 สินค้า / ราคา / Profile

| Mer C Field | → AB Field (เว็บ) | Excel cell (C To AB) |
|-------------|-------------------|----------------------|
| No. of / Material Grouping | `materialGroupingNo` → buy/get material ref | AQ, BI |
| No. of / Bonus Buy | `bonusBuyNo` (เขียนซ้ำหลายฝั่ง) | T, AN, BF, FG, GO |
| Pur Group | `purchasingGroup` | C9 |
| Material | `material` (หรือเข้า `materialGrouping` sheet) | E ใน Material Grouping เริ่มแถว 4 |
| Vendor | `vendorCode` | L12 |
| Sales Unit | `get.unit` (บาง profile มีฝั่ง buy ด้วย) | CE, HA, HP |
| Sales Price / Normal | `get.normalPrice` | BM |
| Sales Price / Promo | `get.promoPrice` | BW |
| Bonus Buy Profile | `bonusBuyProfile` | C10 |
| Discount Amt | `get.discountAmt` | BZ |
| Discount % / For P015 | `get.discountPct` | CA |
| Discount % / PLU | `get.discountPct` | CA |

หมายเหตุฝั่ง Buy (บาง profile เช่น D001 / F001):

| ความหมาย | → AB Field (เว็บ) | Excel cell |
|----------|-------------------|------------|
| Bonus Buy No. (Buy) | `buy.bonusBuyNo` | AN |
| ประเภท MAT / MGP (Buy) | `buy.materialType` | AO |
| รหัสสินค้า/กลุ่ม (Buy) | `buy.materialOrGroup` | AQ |
| Buy Qty | `buy.qty` | AV |
| Bonus Buy No. (Get) | `get.bonusBuyNo` | BF |
| ประเภท MAT / MGP (Get) | `get.materialType` | BG |
| รหัสสินค้า/กลุ่ม (Get) | `get.materialOrGroup` | BI |
| Get Qty | `get.qty` | BP |

### 5.4 Compensate / Charge Back / Contract

| Mer C Field | → AB Field (เว็บ) | Excel cell (C To AB) |
|-------------|-------------------|----------------------|
| Compensate / ฿/Qty In SAP | `compensate.bahtPerQty` | HM |
| Compensate / ฿/Set | `compensate.bahtPerSet` | HM |
| Compensate / % | `compensate.percent` | HM |
| Charge Back / Reason | `rebateChargeback` | C12 |
| Charge Back / Contract Type | `contractType` | E12 |
| Incl. or Excl. ZBL - Biglot Sales | `biglotSalesInclExcl` | Gi |
| Charge Back / Settlement Option | `settlementOption` | FE |
| Charge Back / Settlement Plant | `settlementPlant` | FF |
| Charge Back / Payment Terms | `paymentTerms` | EX |
| Charge Back / Payment Method | `paymentMethod` | EY |
| Charge Back / Condition Table | `conditionTable` | HD |
| Charge Back / Field Combination | `fieldCombination` | GG & GH |

> Compensate สามช่องใน Mer C ไป cell เดียวกัน (HM) — บนเว็บเก็บแยกชื่อ แล้วตอน export เลือกค่าที่กรอกตามกฎ convert

### 5.5 ตัวอย่าง DTO สั้นๆ (เว็บ)

```
PromotionAB
  promotionName
  purchasingGroup
  theme
  bonusBuyProfile
  rebateChargeback?
  contractType?
  vendorCode
  periodStart
  periodEnd
  days { all?, mon…sun? }
  plants[]                  // "13KA", "30KA", …
  wbsNo
  lines[]
    bonusBuyNo
    validTimeFrom?
    validTimeTo?
    buy? { materialType, materialOrGroup, qty, unit? }
    get  { materialType, materialOrGroup, qty, normalPrice?,
           promoPrice?, discountAmt?, discountPct?, unit }
  contract?
    compensate?
    settlementOption?
    settlementPlant?
    paymentTerms?
    paymentMethod?
    conditionTable?
    fieldCombination?
    biglotSalesInclExcl?
```

Logic เว็บ map ด้วยชื่อด้านบน; ถ้าต้องได้ไฟล์ Excel AB ค่อย map ชื่อ → cell ตามคอลัมน์ขวาสุด
