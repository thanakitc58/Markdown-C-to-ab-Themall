# Mer C → STD Field Mapping Guide

คู่มือแกะฟิลด์จาก `Mer-C_Convert_To_STD.txt` เพื่อเทียบกับ template จริง และเตรียมสเปกให้เขียนเว็บต่อ

**แหล่งที่มา:** VBA constants + logic ใน `Mer-C_Convert_To_STD.txt`  
**สถานะ:**  
- Mer C V3.7 เทียบแล้ว (ชั้น 1 ผ่าน — ข้อ 0)  
- C2AB ดูเฉพาะ sheet `Promotion BBY` แล้ว (ข้อ 0.1)

---

## 0. ผลการเทียบ Mer C V3.7 (อัตโนมัติ)

เทียบเมื่อ: 2026-08-07  
ไฟล์: `Mer C_Promotion Template_V3.7.xlsb` → `Mer C Template`  
ชีตอื่นในไฟล์: `Mechanic`, `Mapping`, `Mat_MSTR`, `IM Mail Lists`

### สรุป

| หมวด | ผล |
|------|-----|
| Header cells (C4, M4, วันที่, เวลา, WBS, วันในสัปดาห์, Prepared_by) | **ตรง** — label อยู่ช่องข้างๆ/แถวบน ค่าตาม cell ในโค้ด |
| คอลัมน์ที่ VBA ตั้งชื่อ (ยกเว้น Online_EN/TH) | **ตรง** กับหัวคอลัมน์จริง |
| `Online_EN` (138) / `Online_TH` (139) | **ไม่มีใน template นี้** — col 138 = Bonus Buy Text, 139 ว่าง (ตรง comment ในโค้ด) |
| `ConvertStatus` (149) | **ตรง** = Convert Status |
| Plant row 17, data row 19 | **ตรง** |
| คอลัมน์ที่โค้ดไม่ได้ตั้งชื่อ | มีหลายช่อง (ดูตารางช่องว่างด้านล่าง) — ถูก copy ไปด้วยช่วง `B:EQ` |

**ชั้น 1 (ครบตามที่โค้ดใช้):** ผ่าน  
**ขั้นถัดไป:** เจาะ 1 profile บน `Promotion BBY` + golden test

---

## 0.1 Mer C → Promotion BBY (C2AB)

เทียบเมื่อ: 2026-08-07  
ไฟล์: `Convert_C_to_AB_Template_V2.xlsb` → **เฉพาะ sheet `Promotion BBY`**  
(ชีตอื่นใน C2AB ยังไม่ไล่ในรอบนี้ ตามที่กำหนด)

โครงสร้างชีตโดยย่อ:
- แถวบน = **Promotion Header** (ชื่อโปรโม, profile, vendor, วันที่, plant, วันในสัปดาห์)
- ตั้งแต่แถว ~14 เป็นต้นไป = **Bonus Buy lines** (header คอลัมน์อยู่แถว 11–13)
  - คอลัมน์ `T`… = Bonus Buy Header
  - คอลัมน์ `AN`… = **Buy (ซื้อ)**
  - คอลัมน์ `BF`… = **Get (ได้รับ)** ← Gen_P001/P010 ส่วนใหญ่เขียนฝั่งนี้

### A) Header mapping (`Fill_BBY_Header` + `Mark_Plant`)

| Promotion BBY | Label ในชีต | มาจาก Mer C | หมายเหตุ |
|---------------|-------------|-------------|----------|
| C8 | Promotion Name (B8) | Promo_Name `M4` | |
| C9 | Purchasing Group (B9) | PurGroup col 31 | แปลงเป็นชื่อเต็มผ่าน Sub_condition |
| F9 | Theme (E9) | Theme `C4` | ถ้ายาว 4 ตัว อาจ lookup Sub_condition |
| C10 | Bonus Buy Profile (B10) | BBY_Profile col 53 | แปลงเป็นชื่อเต็มผ่าน profile master |
| C12 | Rebate Chargeback (B12) | ChargeBack_Reason → Rebate_Reason | ตัดตัวอักษรแรก / เติม 0 ตามกฎ |
| E12 | Contract Type (D12) | ContractType col 70 | ใส่เมื่อมีสัญญา (เช่น Z2) |
| L12 | Vendor Code (J12) | Vendor col 36 | ว่าง → `NOBP` |
| P11 | START (O11) | วันเริ่ม C7/D7/E7 | รูปแบบวันที่โปรโม |
| P12 | END (O12) | วันจบ G7/H7/I7 | |
| P17 | ALL (O17) | All_Day `M7` | ใช้เมื่อ Day require ≠ N/A |
| P19–P25 | MON…SUN (O19–O25) | Mon–Sun `N7–T7` | ใช้เมื่อ All_Day ว่าง |
| E17 | Online Plant (B17) | plant `22KA` | ติ๊ก X |
| E23 | ALL Offline (B23) | `KA_EXCEPT_ONLINE` | ติ๊ก X |
| E25–E42 | The Mall / Flagship / Stand Alone… | plant จาก Mer C col 3–25 | เช่น 13KA→E25, 14KA→E26, … 66KA→E42 |
| E47 | DC (B47 / 91) | เคส DC | ติ๊ก X |
| E48 | FDC (B48 / 92) | เคส FDC | ติ๊ก X |

### B) Bonus Buy line — Header คอลัมน์ (แถวข้อมูลเริ่มประมาณแถว 14+)

เขียนโดย `P001_Header` / `P010_Header` ฯลฯ ที่ `Bonus_Buy_Row_Index`

| BBY Col | หัวคอลัมน์ใน Promotion BBY | มาจาก / ความหมาย |
|---------|----------------------------|-------------------|
| T | Bonus buy No. | เลข BBY ที่ gen (รันนิ่ง) |
| U | Description | prefix + คำอธิบายในแถว (อ่านแล้วต่อหน้า) |
| W | Valid Time from | Promo_start_Time `C9` |
| X | Valid Time to | Promo_end_Time `F9` |
| AA | Promotion Area | จาก logic (เช่น offline P1 / DC) |
| AB | WBS No. | WBS `M9` |
| AI | Reference code | บาง profile (เช่น P010) |
| AK | Online Description EN | Online_EN (ใน Mer C V3.7 ไม่มีที่ col 138) |
| AL | Online Description TH | Online_TH (เช่นกัน) |

### C) Get side (ได้รับ) — ที่ Gen_P001 / P010 ใช้บ่อย

| BBY Col | หัวคอลัมน์ | มาจาก Mer C | กฎสั้นๆ |
|---------|-----------|-------------|---------|
| BF | Bonus Buy No. (Get) | เลข BBY เดียวกับ T | |
| BG | ประเภท Material / Material Group | — | มี Mat_Group → `MGPNew - New Material Group` / ไม่มี → `MAT - Material` |
| BI | รหัสสินค้าหรือกลุ่ม | Mat_no col 32 หรือชื่อ Mat_Group | ตาม BG |
| BM | ราคาขาย (Normal) | SalePrice_Normal col 51 | |
| BP | Get Qty | Mechanic col 54 | lookup sheet Mechanic → Get qty |
| BW | ราคาจัดรายการ [P] | SalesPrice_Promo col 52 | ถ้ามีค่า ใช้ก่อน |
| BZ | ส่วนลดบาท [R] | Discount_Amt col 55 | ถ้าไม่มี New Price |
| CA | ส่วนลด % | DiscountPercent_P015/PLU col 56/58 | ถ้าไม่มีบาท; มี normalize % |
| CE | Unit | SalesUnit col 40 | |

ลำดับส่วนลดบน Get (ตรงโค้ด): **Promo price → Discount Amt → Discount %**

### D) Buy side (ซื้อ) — ใช้บาง profile (เช่น D001)

| BBY Col | หัวคอลัมน์ | บทบาทคล้าย Get |
|---------|-----------|----------------|
| AN | Bonus Buy No. (Buy) | เลข BBY |
| AO | ประเภท MAT/MGP | เหมือน BG |
| AQ | รหัสสินค้า/กลุ่ม | เหมือน BI |

(รายละเอียด Buy เพิ่มตาม profile — ยังไม่เจาะทุก Gen ในรอบนี้)

### E) สิ่งที่ชีตนี้มี แต่รอบนี้ยังไม่ map ครบ

ชีต `Promotion BBY` กว้างมาก (Card type, Tender, Contract block คอลัมน์ ES… ฯลฯ)  
โค้ด VBA เขียน contract หนักใน `Fill_Contract` — **อยู่นอก scope รอบนี้** ถ้าโฟกัสแค่ header + Get line ของ P001/P010

### Checklist ใช้ Promotion BBY ต่อ

- [x] รู้ cell header ที่ VBA เขียน
- [x] รู้คอลัมน์ Get หลัก (BF–CE) เทียบกับ Mer C
- [ ] เจาะ 1 profile (แนะนำ P001 หรือ P010) ทีละแถวว่าเขียนครบไหม
- [ ] golden test: Mer C ตัวอย่าง → แถวบน Promotion BBY ตรงตารางด้านบนไหม

---

## 1. ภาพรวมระบบ

สคริปต์นี้เป็น VBA ของ Excel macro สำหรับแปลงไฟล์โปรโมชัน **Mer C** ให้เป็นรูปแบบ **STD / Bonus Buy (BBY)** ที่ระบบ RPA/SAP ใช้ต่อได้

| ฝั่ง | ไฟล์ / Sheet | บทบาท |
|------|----------------|--------|
| Input | `Mer C_Promotion Template_V3.7.xlsb` → sheet `Mer C Template` | ต้นทางข้อมูลโปรโม |
| Template แปลง | `Convert_C_to_AB_Template_V2.xlsb` (C2AB) | ปลายทาง + master data |
| Output sheets ใน C2AB | `Promotion BBY`, `(2)Create-Data`, `Mechanic`, `Material Grouping`, `Sub_condition` | โครงสร้าง STD/BBY |
| Log / ส่งต่อ | SQL `RPA_PRD_DB` + โฟลเดอร์ APPROVED | หลัง convert สำเร็จ |

### Pipeline หลัก

```
Upload Mer C
  → อ่าน header (ชื่อโปรโม / วันที่ / WBS / วันในสัปดาห์)
  → แบ่งกลุ่มตาม Number_of_Promotion
  → skip: Reject / Done / On Pack (B1G1, B2G1)
  → copy ช่วงแถวไป template C2AB
  → แยกเคส: Online only / Offline+Online / DC / Contract(Z2/Z3)
  → gen ตาม BBY_Profile (P001, P010, …)
  → ตั้งชื่อไฟล์ + save
  → log SQL + copy ไป approve folder
```

Entry point ในโค้ด: `Convert_to_STD_Link` → `Search_Promotion` → `Copy_Mer_C_Data`

---

## 2. ไฟล์ใน Drive ดูอันไหน

จากโฟลเดอร์ `0 - All Merchandise Template`:

| ลำดับ | ไฟล์ | ใช้ทำอะไร |
|------|------|-----------|
| 1 (เริ่มที่นี่) | `Mer C_Promotion Template_V3.7.xlsb` | เทียบฟิลด์ต้นทางกับตารางในเอกสารนี้ |
| 2 | `Convert_C_to_AB_Template_V2.xlsb` | เทียบปลายทาง + master sheets (ตรงชื่อในโค้ด) |
| ยังไม่ต้อง | `(ส่งซัพพลายเออร์ Grocery)_...` | ฟอร์มซัพพลายเออร์ คนละเรื่อง |
| ยังไม่ต้อง | `Convert_Macro_20230818.xlsm` | macro เก่า — มี logic ใน `.txt` แล้ว |
| ดูทีหลัง | `Promotion_..._MerAB_V4.38.xlsm` | สาย MerAB / เวอร์ชันใหม่กว่า |

**กฎสั้นๆ**
- เทียบฟิลด์ต้นทาง → ใช้ **Mer C**
- เทียบว่าค่าไปโผล่ช่องไหนหลังแปลง → ใช้ **C2AB / AB**
- อย่าเริ่มที่ MerAB ถ้ายังไม่ได้ล็อก mapping ของ Mer C + C2AB ตามโค้ดชุดนี้

---

## 3. สิ่งที่เอาไปเทียบกับ Mer C Template

เอาเฉพาะรายการด้านล่าง (ต้นทาง) ไปเทียบกับ `Mer C Template`  
**ยังไม่ต้อง** เอา cell ปลายทางแบบ BBY `C8`, `BF`, `BW` ไปเทียบ Mer C

### 3.1 โครงแถว / Sheet

| Const | ค่า | ความหมาย |
|-------|-----|----------|
| sheet name | `Mer C Template` | ชีตต้นทาง |
| `row_plant_no` | 17 | แถวรหัส plant |
| `row_start` | 19 | แถวเริ่มข้อมูลสินค้า |
| plant columns | 3–25 | ติ๊กสาขา (X) |
| copy range | `B:EQ` (จาก first→last promo row) | copy กว้างกว่า constants ที่ตั้งชื่อ |

### 3.2 Header (Cell Address)

| Const | Cell | ความหมาย | ผลเทียบ | หลักฐานใน Mer C V3.7 |
|-------|------|----------|---------|----------------------|
| Theme | C4 | ธีมโปรโม | ตรง | label `Theme :` ที่ B4 |
| Promo_Name | M4 | ชื่อโปรโม | ตรง | label `Promotion Name` ที่ M3 |
| Prepared_by | C12 | คนเตรียม | ตรง | label `Prepared by:` ที่ C11 |
| Promo_start_day | C7 | วันเริ่ม | ตรง | กลุ่ม Period ที่ B7 |
| Promo_start_month | D7 | เดือนเริ่ม | ตรง | กลุ่ม Period |
| Promo_start_year | E7 | ปีเริ่ม | ตรง | กลุ่ม Period; hint `DD\|MM\|YYYY` ที่ F6 |
| Promo_end_day | G7 | วันจบ | ตรง | หลัง `To` ที่ F7 |
| Promo_end_month | H7 | เดือนจบ | ตรง | |
| Promo_end_year | I7 | ปีจบ | ตรง | |
| Promo_start_Time | C9 | เวลาเริ่ม | ตรง | label `Time :` ที่ B9 |
| Promo_end_Time | F9 | เวลาจบ | ตรง | หลัง `To` ที่ E9 |
| WBS_no | M9 | WBS | ตรง | label `WBS No.` ที่ L9 |
| All_Day | M7 | ทั้งวัน? | ตรง | label `All` ที่ M6 (ค่าตัวอย่างใน template = X) |
| Mon | N7 | จันทร์ | ตรง | label ที่ N6 |
| Tue | O7 | อังคาร | ตรง | O6 |
| Wed | P7 | พุธ | ตรง | P6 |
| Thu | Q7 | พฤหัส | ตรง | Q6 |
| Fri | R7 | ศุกร์ | ตรง | R6 |
| Sat | S7 | เสาร์ | ตรง | S6 |
| Sun | T7 | อาทิตย์ | ตรง | T6 |

**ความสัมพันธ์ header:** `All_Day` **หรือ** `Mon–Sun` (เลือกชุดหนึ่ง) + วันที่เริ่ม/จบ + เวลา + WBS

### 3.3 คอลัมน์รายการสินค้า (Column Number)

| Const | Col | ความหมาย | ผลเทียบ | หัวคอลัมน์จริงใน Mer C V3.7 |
|-------|-----|----------|---------|------------------------------|
| plant_start … plant_end | 3–25 | สาขา / ติ๊ก X | ตรง | C–Y เป็นรหัสร้าน (13KA… / XX KA); แถว 17 |
| OnlineStatus | 26 | สถานะ | ตรง | Status |
| Number_of_Promotion | 27 | เลขกลุ่มโปรโม | ตรง | No. of \| Promotion |
| Mat_Group | 28 | Material Group | ตรง | Material Grouping |
| BBY_no | 29 | เลข BBY (ระบบเขียนกลับ) | ตรง | Bonus Buy |
| MCH_LV2 | 30 | Merchandise hierarchy | ตรง | MCH |
| PurGroup | 31 | Purchasing Group | ตรง | Pur Group |
| Mat_no | 32 | Material Number | ตรง | Material |
| Barcode | 33 | Barcode | ตรง | Barcode |
| *(ช่องว่างใน constants)* | 34–35 | ไม่ได้ตั้งชื่อในโค้ด | มีในไฟล์ | 34 Material (TH) Des. / 35 Material (EN) Des. |
| Vendor | 36 | Vendor (ว่าง → `NOBP`) | ตรง | Vendor |
| *(ช่องว่าง)* | 37 | ไม่ได้ตั้งชื่อ | มีในไฟล์ | Vendor Des. |
| FlowType | 38 | Flow Type | ตรง | Flow Type |
| *(ช่องว่าง)* | 39 | ไม่ได้ตั้งชื่อ | มีในไฟล์ | Pack Size |
| SalesUnit | 40 | หน่วยขาย | ตรง | Sales Unit |
| *(ช่องว่าง)* | 41–50 | ไม่ได้ตั้งชื่อ | มีในไฟล์ | ดูตารางช่องว่างด้านล่าง |
| SalePrice_Normal | 51 | ราคาปกติ | ตรง | Sales Price \| Normal |
| SalesPrice_Promo | 52 | ราคาโปร (New Price) | ตรง | Promo (ใต้ Sales Price) |
| BBY_Profile | 53 | Profile เช่น P001, P010 | ตรง | Bonus Buy Profile |
| Mechanic | 54 | Mechanic เช่น B1G1 | ตรง | Mechanic |
| Discount_Amt | 55 | ส่วนลดบาท | ตรง | Discount Amt |
| DiscountPercent_P015 | 56 | ส่วนลด % (P015) | ตรง | Discount % \| For P015 |
| *(ช่องว่าง)* | 57 | ไม่ได้ตั้งชื่อ | มีในไฟล์ | Deal |
| DiscountPercent_PLU | 58 | ส่วนลด % PLU | ตรง | PLU (ในกลุ่ม Discount %) |
| Compensate_BahtPerQty_SAP | 59 | ชดเชยบาท/ชิ้น | ตรง | Compensate \| ฿/Qty In SAP |
| Compensate_BahtPerSet | 60 | ชดเชยบาท/ชุด | ตรง | ฿/Set |
| Compensate_Percent | 61 | ชดเชย % | ตรง | % (ใต้กลุ่ม Compensate) |
| *(ช่องว่าง)* | 62–65 | ไม่ได้ตั้งชื่อ | มีในไฟล์ | GP% Normal/Promo, Mix, Media |
| PromoTag | 66 | Promo Tag | ตรง | Promo Tag |
| *(ช่องว่าง)* | 67–68 | ไม่ได้ตั้งชื่อ | มีในไฟล์ | Remind, Remarks |
| ChargeBack_Reason | 69 | เหตุผล chargeback | ตรง | Charge Back \| Reason |
| ContractType | 70 | ประเภทสัญญา (Z2/Z3…) | ตรง | Contract Type |
| ChargeBack_ContractType_Inc_Exc | 71 | Inc/Exc | ตรง | Contract Type Z200/Z222 Incl. or Excl. … |
| ChargeBack_SettlementOption | 72 | Settlement option | ตรง | Settlement Option |
| ChargeBack_SettlementPlant | 73 | Settlement plant | ตรง | Settlement Plant |
| PaymentTerms | 74 | Payment terms | ตรง | Payment Terms |
| PaymentMethod | 75 | Payment method | ตรง | Payment Method |
| ConditionTable | 76 | Condition table | ตรง | Condition Table |
| FieldCombination | 77 | Field combination | ตรง | Field Combination |
| IM_Reject | 78 | Reject หรือไม่ | ตรง | IM \| Reject |
| Online_EN | 138 | ข้อความ online EN | **ไม่มี** | จริง = Bonus Buy Text |
| Online_TH | 139 | ข้อความ online TH | **ไม่มี** | ว่าง |
| ConvertStatus | 149 | Done หรือยัง | ตรง | Convert Status |

### 3.4 คอลัมน์ที่มีใน Mer C แต่โค้ดไม่ได้ตั้งชื่อ (ช่องว่าง)

ถูก copy ไป C2AB ผ่านช่วง `B:EQ` แต่ไม่มี business rule โดยตรงใน constants:

| Col | หัวคอลัมน์จริง |
|-----|----------------|
| 34 | Material (TH) Des. |
| 35 | Material (EN) Des. |
| 37 | Vendor Des. |
| 39 | Pack Size |
| 41 | Sales Tax |
| 42 | Fix GP |
| 43 | Cost Price Case (Exc.Vat) Normal |
| 44–46 | Discount Amt / % / FOC (ฝั่ง cost) |
| 47–48 | Date Start / End |
| 49–50 | Unit (Inc.Vat) Normal / Promo |
| 57 | Deal |
| 62–65 | GP % Normal / Promo / Mix / Media |
| 67–68 | Remind / Remarks |
| 79+ | Reason, Forecast, Sales by month/store, Display, sorting fields ฯลฯ |

> หมายเหตุ: โค้ดใช้ `col_no_plant_end = 25` เป็นคอลัมน์เช็ค Online plant ด้วย — ใน template col 25 เป็น `XX KA` (plant placeholder) ส่วน col 26 = Status

---

## 4. กลุ่มฟิลด์และความสัมพันธ์

### A) Header โปรโมชัน
Theme, Promo_Name, Prepared_by, วันที่, เวลา, WBS, All_Day / Mon–Sun  
→ ใช้ร่วมทุกแถวในไฟล์

### B) ตัวแบ่งกลุ่ม / สถานะแถว

| Field | บทบาท |
|-------|--------|
| Number_of_Promotion | ตัดช่วง first→last แล้ว convert ทีละกลุ่ม |
| IM_Reject | `Reject` → ข้ามแถว |
| ConvertStatus | `Done` → ไม่แปลงซ้ำ |
| Mechanic = B1G1/B2G1 (On Pack) | skip ทั้งกลุ่ม |

### C) Plant / Online
- Col 3–24 = offline plants  
- Col ท้ายโซน plant = online  
- Online only → ไม่เข้า DC  
- Offline+Online → แยกเคส  
- DC → plant บังคับประมาณ `91KA`

### D) ตัวตนสินค้า
- มี `Mat_Group` → ใช้กลุ่ม + `MCH_LV2` + `SalesUnit` (แบบ MGP)  
- ไม่มี `Mat_Group` → ใช้ `Mat_no` + `Barcode` + `SalesUnit`  
- `Vendor` ว่าง → `"NOBP"`  
- `PurGroup` → lookup ชื่อเต็มใน Sub_condition

### E) ราคา / ส่วนลด (ลำดับเลือกใน Gen)

```
SalesPrice_Promo (New Price)
  else Discount_Amt
  else DiscountPercent_* (P015 หรือ PLU ตาม Profile + ความยาว Barcode)
+ SalePrice_Normal + SalesUnit + Mechanic (Buy/Get)
```

### F) Profile + Mechanic
- `BBY_Profile` = ชนิดโครงสร้าง / เลือก `Gen_P001`, `Gen_P010`, …  
- `Mechanic` = จำนวนซื้อ–แถม (lookup sheet Mechanic)

Profiles ที่พบในโค้ด: `P001`, `P010`, `P011`, `P015`, `P100/P103`, `D001`–`D003`, `F001`–`F003`

### G) PromoTag + Compensate + ContractType (แยกเคสไฟล์)

```
ContractType ขึ้นต้น Z2 → is_Compensate = true
ContractType ขึ้นต้น Z3 → isZ3 = true

แยกไฟล์ตาม combo:
  มี PromoTag + มี Compensate(+Z2)
  มี PromoTag + ไม่มี Compensate / เป็น Z3
  ไม่มี PromoTag + มี Compensate(+Z2)
  ไม่มี PromoTag + ไม่มี Compensate / เป็น Z3
```

ชื่อไฟล์ตัวอย่างแนว:  
`YYYYMMDD_P010_C_BBY_...` หรือ `..._C_BBYCT_...` + PurchasingGroup + Vendor + timestamp + Prepared_by

### H) Chargeback / Contract block
ChargeBack_Reason, ContractType, Inc/Exc, SettlementOption/Plant, PaymentTerms/Method, ConditionTable, FieldCombination  
→ ใช้หนักใน `Fill_Contract` เมื่อมีสัญญา

### โครง DTO แนะนำสำหรับเว็บ

```
PromotionHeader          ← กลุ่ม A
PromotionGroup           ← Number_of_Promotion + skip rules (B)
  Channel/Plant          ← กลุ่ม C
  LineItems[]            ← D + E
  RoutingKey             ← Profile + Mechanic + PromoTag + Contract/Compensate (F+G)
  ContractBlock?         ← กลุ่ม H
```

---

## 5. วิธีเทียบกับ Mer C (ทำตามนี้)

1. เปิด `Mer C_Promotion Template_V3.7.xlsb` → ชีต `Mer C Template`
2. เช็ค header ตามตาราง Cell ในข้อ 3.2
3. เดินทีละคอลัมน์ตามข้อ 3.3: ดูหัวคอลัมน์จริง แล้วติ๊กผล
4. จดสถานะต่อช่องเป็นอย่างใดอย่างหนึ่ง:
   - **ตรง** — เลข/ความหมายตรงโค้ด
   - **เลื่อน** — ความหมายเดิมแต่คนละเลขคอลัมน์ (จดเลขใหม่)
   - **ไม่มี** — ใน template ไม่มี (เช่น Online_EN/TH)
5. เดินหัวคอลัมน์ที่โค้ดไม่ได้ตั้งชื่อ (ช่องว่าง) แล้วจดว่ามีฟิลด์อะไร — สำคัญต่อชั้นที่ 2 ด้านล่าง

---

## 6. จะรู้ได้ไงว่าครบ

### ชั้น 1 — ครบตามที่โค้ดใช้ (ขั้นต่ำ)
ทุกแถวในตารางข้อ 3.2 และ 3.3 มีสถานะ ตรง / เลื่อน / ไม่มี แล้ว  
→ mapping ที่ VBA อ้างอิงครบ

### ชั้น 2 — ครบตาม template จริง
ไล่หัวคอลัมน์ใน Mer C ทั้งแถว แล้วถามว่า “คอลัมน์นี้มีในเอกสารนี้ไหม?”
- มีในโค้ด → OK
- ไม่มีในโค้ด → ตัดสินใจว่า:
  - แค่ถูก copy ไป AB (`B:EQ`) → อาจยังไม่ต้องมี business rule
  - ธุรกิจใช้ตัดสินใจ/แสดงผล → ต้องเพิ่มเข้า dictionary

### ชั้น 3 — ครบแบบพิสูจน์ด้วยเคสจริง
1. ใช้ไฟล์ Mer C ที่เคย convert สำเร็จ
2. ดูว่าฟิลด์ไหนมีค่า
3. เปิด output / C2AB ดูว่าค่าไปโผล่ช่องไหน
4. ทุกค่าสำคัญอธิบายได้ = พร้อมส่งเพื่อนเขียนเว็บ

> ตารางในเอกสารนี้ **ยังไม่ใช่ data dictionary สมบูรณ์** จนกว่าจะผ่านอย่างน้อยชั้น 1 + ทด 1 เคสจริง

---

## 7. สิ่งที่ยังไม่ครบจากโค้ดอย่างเดียว

1. Mer C มีคอลัมน์มากกว่าที่ตั้งชื่อใน constants (เพราะ copy ช่วง `B:EQ`)
2. เลขคอลัมน์เคยเลื่อนหลายรอบ — ต้องยืนยันกับ template ปัจจุบัน
3. ฝั่ง AB ปลายทาง (cell `C8`, `BF`, `BW` ฯลฯ ใน `Gen_Pxxx`) ยังไม่ได้ map ครบในเอกสารนี้
4. บางฟิลด์อาจเลิกใช้แล้ว (`Online_EN` / `Online_TH`)
5. Master data อยู่ใน C2AB (`Mechanic`, `Sub_condition`, profile master) ไม่ได้อยู่ใน Mer C

---

## 8. Checklist ส่งเพื่อนเขียนเว็บ

- [ ] Flowchart pipeline (ข้อ 1)
- [ ] Field mapping Mer C หลังเทียบ template แล้ว (ข้อ 3 — แก้เลขให้ตรงจริง)
- [ ] กลุ่มความสัมพันธ์ / DTO (ข้อ 4)
- [ ] ตาราง routing ตาม Profile (เริ่มจาก profile ที่ใช้บ่อย)
- [ ] Sample Mer C input + expected output อย่างน้อย 1 เคส
- [ ] ระบุ out of scope รอบแรก (เช่น On Pack, ทุก profile, copy SFTP เดิม)
- [ ] อย่าพอร์ต SQL password จาก VBA ตรงๆ — ใช้ env/secret

### กับดักที่ควรรู้
- โค้ด VBA ซ้ำหลาย branch (Online / Contract / DC) — บนเว็บควรรวมเป็น strategy ตาม profile
- ล็อกเวอร์ชัน template ให้ชัด (Mer C V3.7 + C2AB V2 ตามชุดนี้)
- `Sub_condition` / Purchasing Group / Profile full name = master data → ย้ายเป็น DB/API

---

## 9. ขั้นถัดไป

1. ~~อัปเดตตารางข้อ 3 ให้เป็นเลขคอลัมน์จริง~~ ✅ Mer C V3.7
2. ~~map ปลายทาง Promotion BBY (header + Get)~~ ✅ ข้อ 0.1
3. เจาะ 1 profile บน Promotion BBY (แนะนำ P001 หรือ P010)
4. ทำ golden test: Mer C input → แถวบน/แถว Get ใน Promotion BBY
5. ส่งเพื่อนเป็นสเปกเว็บ: `parseMerC` → `convertPromotion` → `exportToExcel` (โฟกัส sheet Promotion BBY ก่อน)
6. บนเว็บ: ไม่พึ่ง Mer C col 138/139 เป็น Online EN/TH (ใน V3.7 ไม่ใช่ช่องนั้น)

---

## อ้างอิงในโค้ด

| หัวข้อ | ตำแหน่งโดยประมาณใน `Mer-C_Convert_To_STD.txt` |
|--------|-----------------------------------------------|
| Constants คอลัมน์/cell | บรรทัด 1–62 |
| C2AB sheet names | 82–97 |
| `Convert_to_STD_Link` | 99 |
| `Search_Promotion` | 128 |
| `set_RebateReason_n_ContractType` | 400 |
| `Copy_Mer_C_Data` | 458 |
| `Fill_BBY_Header` | 3063 |
| `Gen_P001` และ profile อื่นๆ | 3620+ |
| `Fill_Contract` | 4625 |
| `Copy_From_Input_to_STD` | 6628 |
| SQL log / copy approve | 6815–6848 |
