# 03 — `CONDITIONS[]` (สัญญา)

> อ่าน [README.md](README.md) ก่อน (หลักการ + กฎ Autofill มาก่อน Lookup)
> ไฟล์ lookup ที่ใช้:
> - [lookup/plant-sales-org.json](lookup/plant-sales-org.json) — Plant → Sales Organization
> - [lookup/mechanic.json](lookup/mechanic.json) — QTY/SET (`qty_set`)

ตอนนี้ convert **pass-through** ตามที่ user กรอก → ต้องเปลี่ยนเป็นสร้างค่าตาม macro (`Fill_Contract`) ดังนี้

---

## 1. Autofill มาก่อน Lookup (เฉพาะไฟล์นี้)

| ค่า | ใช้ค่า autofill จาก input ก่อน | ถ้าว่าง → lookup |
|---|---|---|
| `sales_organization` (Settlement Option `1`) | `conditionHeader.sales_organization` | [plant-sales-org.json](lookup/plant-sales-org.json) ค้นด้วย settlement plant |
| `quantity` (`Z222`) | `conditionHeader.quantity` | [mechanic.json](lookup/mechanic.json) → `qty_set` ค้นด้วย Mechanic |
| `condition_rate` (ZR05) | `MATERIALS[].compensate_quantity_in_sap` | `MATERIALS[].compensate_baht_per_set` |
| `vendor_name`, `department_mer`, `start`, `end`, `department`, `set_of_field_combination`, `text_field_combination`, `selection_group`, `condition_unit` | ค่าใน `CONDITIONS[]` ของ input | – |

> ⚠️ ข้อยกเว้น: Settlement Option `2` ที่ต้องแตกเป็นหลายสัญญาตาม Sales Org — `sales_organization` ของแต่ละสัญญาต้องมาจาก [plant-sales-org.json](lookup/plant-sales-org.json) เพราะหน้าเว็บมีค่าเดียวต่อแถว

---

## 2. สร้างสัญญาเมื่อไหร่

- มีแถวไหนก็ได้ในโปรที่ Contract Type ขึ้นต้น `Z2` → สร้างสัญญา
- Contract Type ว่าง หรือขึ้นต้น `Z3` → **ไม่ส่ง** `CONDITIONS[]` (Z3 แค่เติม prefix `CPS3_` ใน description)
- BBY ของ DC → ไม่สร้างสัญญา

---

## 3. Settlement Option และจำนวนสัญญา

ทำก่อนข้ออื่น เพราะกำหนดว่าต้องสร้างกี่สัญญา

| เงื่อนไข (เช็คตามลำดับ) | `settlement_option` | `settlement_plant` |
|---|---|---|
| Plant online (`22KA`) | `2 - Settlement to same business volume plant with in company` | ว่าง |
| มีแถว Flow = `DC` หรือ `XD` | `1 - Settlement to Settlement Plant` | `91KA` |
| แถวแรกกรอก Settlement Option ขึ้นต้น `1` | `1 - Settlement to Settlement Plant` | ตามที่กรอกในแถว |
| อื่นๆ (ว่าง หรือขึ้นต้น `2`) | `2 - Settlement to same business volume plant with in company` | ว่าง |

จำนวนสัญญา (`contract_number` รันเลข 1, 2, 3 …):

| Settlement Option | สร้างสัญญา | `sales_organization` |
|---|---|---|
| `1` | **1 สัญญาต่อ 1 BBY** | autofill ก่อน · ถ้าว่างใช้ Sales Org ของ settlement plant ([plant-sales-org.json](lookup/plant-sales-org.json)) |
| `2` | **1 สัญญาต่อ 1 BBY ต่อ 1 Sales Org** | `stores` = `KA_EXCEPT_ONLINE` → ครบ 7 Sales Org (= 7 สัญญาต่อ BBY) · อื่นๆ → Sales Org ไม่ซ้ำของสาขาที่เลือก (lookup เสมอ) |

---

## 4. `conditionHeader` (1 ต่อ 1 สัญญา)

| key | ค่าตาม macro |
|---|---|
| `contract_number` | เลขสัญญาตามข้อ 3 |
| `settlement_option` / `settlement_plant` / `sales_organization` | ตามข้อ 3 |
| `payment_term` / `payment_method` | จากแถว (Option 1: แถวแรกของ BBY นั้น · Option 2: แถวแรกของโปร) |
| `additional_text` | `HEADER.promotionName` ⚠️ **ไม่ใช่** `header_text` |
| `auto_allocation` | `X` **เฉพาะ Option 1** · Option 2 ว่าง |
| `bonus_buy_id` | BBY No — **เฉพาะ** Contract Type `Z222` |
| `quantity` | **เฉพาะ** `Z222`: autofill ก่อน · ถ้าว่างใช้ `qty_set` ใน [mechanic.json](lookup/mechanic.json) · Mechanic ที่ `qty_set` เป็น `null` → ส่งว่าง |
| `reason`, `contract_type`, `vendor`, `vendor_name`, `department_mer`, `start`, `end`, `department`, `header_text` | macro ไม่ได้แก้ → ใช้ค่าจาก input เดิม |

---

## 5. `conditionType[]`

`condition_table` = 5 ตัวแรกของ Condition Table ในแถวแรก (เช่น `V 163`)

| Condition Table | จำนวนแถว |
|---|---|
| `V 163` (เก็บรายสินค้า) | 1 แถวต่อ 1 สินค้าใน BBY · ใส่ `material` และ `unit` = `sales_unit` |
| อื่นๆ เช่น `V 4AB` (เก็บทั้งสัญญา) | **1 แถวต่อ 1 สัญญา** ไม่ใส่ `material` |

| key | กฎ |
|---|---|
| `contract_number` | เลขสัญญา |
| `condition_type` | กรอก Compensate `%` → `ZR01 - Charge Back %` · ไม่กรอก → `ZR05 - Charge Amount(PeQty)` |
| `condition_rate` (ZR01) | Compensate `%` · ถ้า ≤ 1 คูณ 100 · ถ้า > 100 หาร 100 |
| `condition_rate` (ZR05) | autofill `compensate_quantity_in_sap` (฿/Qty In SAP) · ถ้าว่าง หรือ Contract Type = `Z222` → `compensate_baht_per_set` (฿/Set) |

---

## 6. `businessVolumeSales[]`

`field_combination` = 4 ตัวแรกของ Field Combination ในแถวแรกของ BBY

ต่อ 1 สัญญา สร้างแถวดังนี้ — จำนวนแถว = **จำนวนสินค้า × 2 + จำนวนสินค้า Exclusive** (Z265 คูณจำนวนสาขาอีก)

**(ก) แถวปกติ** — ทุกสินค้าใน BBY × Billing Type 2 ค่า (`ZFP0`, `ZFP2`)

| key | ค่า |
|---|---|
| `contract_number` | เลขสัญญา |
| `field_combination` | ตามแถว |
| `material` / `sales_units` | รหัสสินค้า / `sales_unit` |
| `billing_type` | `ZFP0` หรือ `ZFP2` |
| `bonus_buy` | BBY No · ถ้า Field Combination = `Z265` หรือ `Z256` → **ว่าง** |
| `condition_type` | `ZPN1` เมื่อ Field Combination = `Z265` หรือ `Z256` |
| `bonus_buy_plant` | เฉพาะ `Z265`: สร้างแถวซ้ำ 1 แถวต่อ 1 สาขา (Option 1 = สาขาที่เลือก · Option 2 = สาขาที่อยู่ใน Sales Org ของสัญญานั้น ตาม [plant-sales-org.json](lookup/plant-sales-org.json) ไม่รวม `22KA`, `68KA`) |

**(ข) แถว Exclusive** — เพิ่มเฉพาะสินค้าที่ Inclusive/Exclusive = `Exclusive` (สินค้านั้นยังมีแถวปกติด้วย ตาม macro)

| key | ค่า |
|---|---|
| `billing_type` | `ZFP2` |
| `inclusive_exclusive` | `Exclusive` |
| `billing_usage_zpf2` | `ZBL` ⚠️ key นี้ ไม่ใช่ `billing_usage` |
| `field_combination` | `Z265` → `Z266` · `Z256` → `Z263` |
| `condition_type` / `bonus_buy` / `bonus_buy_plant` | เหมือนแถวปกติ |

ค่า default (`set_of_field_combination`, `text_field_combination`, `selection_group`, `vendor`, `inclusive_exclusive` ของแถวปกติ = `Inclusive`) → ใช้ค่าจาก input เดิม

---

## 7. `settlementCalendar[]`

1 แถวต่อ 1 สัญญา: `contract_number` + `settlement_date` = `HEADER.periodTo` (รูปแบบ `DD.MM.YYYY`)

## 8. `businessVolumePurchase[]`

macro **ไม่สร้าง** (Mer C ไม่มีคอลัมน์นี้) → ไม่ต้องส่ง

---

## 9. สัญญาของ BBY online

BBY online ที่แยกออกมา ([02-BBY.md](02-BBY.md) ข้อ 5) ถ้ามี Contract Type `Z2` → สร้างสัญญาแยกของตัวเอง

| key | ค่า |
|---|---|
| `settlement_option` | `2 - Settlement to same business volume plant with in company` |
| `sales_organization` | `3010` (สาขา `22KA`) |
| `contract_number` | ต่อจากสัญญาของ BBY offline |
| `conditionType[]` (`V 163`) | ใส่เฉพาะสินค้าที่เปิดขาย online |

---

## 10. ตัวอย่าง — มีสัญญา (Contract Type Z2)

Input (สมมติ):
- `HEADER`: Promotion Name `M Price 10/2026`, Period `01.10.2026`–`31.10.2026`
- `MATERIALS[]` BBY 1 มี 2 แถว · Pur Group `C07`, Profile `D001`, Mechanic `2For`, Vendor `SCJ00`, สาขา `13KA` `14KA` (Sales Org `3010` ชุดเดียว)

| แถว | Material | Sales Unit | ฿/Qty In SAP | Inclusive/Exclusive |
|---|---|---|---|---|
| 1 | `1000473882` | `EA` | `7` | Inclusive |
| 2 | `1000473883` | `EA` | `10` | Exclusive |

- `CONDITIONS[]` แถวแรก: Reason `X`, Contract Type `Z204`, Condition Table `V 163`, Field Combination `Z256`, Settlement Option `2`, Payment Method `K`

Output:

```json
{
  "CONDITIONS": [
    {
      "line_number": 1,
      "contract_number": "1",
      "conditionHeader": {
        "contract_number": "1",
        "reason": "X",
        "contract_type": "Z204",
        "vendor": "SCJ00",
        "department_mer": "Mer C",
        "start": "01.10.2026",
        "end": "31.10.2026",
        "payment_method": "K",
        "sales_organization": "3010 - THE MALL GROUP CO.,",
        "settlement_option": "2 - Settlement to same business volume plant with in company",
        "settlement_plant": "",
        "auto_allocation": "",
        "additional_text": "M Price 10/2026",
        "department": "DEP034",
        "...": "..."
      },
      "businessVolumePurchase": [],
      "businessVolumeSales": [
        { "contract_number": "1", "field_combination": "Z256", "inclusive_exclusive": "Inclusive", "material": "1000473882", "sales_units": "EA", "billing_type": "ZFP0", "bonus_buy": "", "condition_type": "ZPN1", "...": "..." },
        { "contract_number": "1", "field_combination": "Z256", "inclusive_exclusive": "Inclusive", "material": "1000473883", "sales_units": "EA", "billing_type": "ZFP0", "bonus_buy": "", "condition_type": "ZPN1", "...": "..." },
        { "contract_number": "1", "field_combination": "Z256", "inclusive_exclusive": "Inclusive", "material": "1000473882", "sales_units": "EA", "billing_type": "ZFP2", "bonus_buy": "", "condition_type": "ZPN1", "...": "..." },
        { "contract_number": "1", "field_combination": "Z256", "inclusive_exclusive": "Inclusive", "material": "1000473883", "sales_units": "EA", "billing_type": "ZFP2", "bonus_buy": "", "condition_type": "ZPN1", "...": "..." },
        { "contract_number": "1", "field_combination": "Z263", "inclusive_exclusive": "Exclusive", "material": "1000473883", "sales_units": "EA", "billing_type": "ZFP2", "billing_usage_zpf2": "ZBL", "bonus_buy": "", "condition_type": "ZPN1", "...": "..." }
      ],
      "conditionType": [
        { "contract_number": "1", "condition_table": "V 163", "condition_type": "ZR05 - Charge Amount(PeQty)", "condition_rate": "7", "material": "1000473882", "unit": "EA", "...": "..." },
        { "contract_number": "1", "condition_table": "V 163", "condition_type": "ZR05 - Charge Amount(PeQty)", "condition_rate": "10", "material": "1000473883", "unit": "EA", "...": "..." }
      ],
      "settlementCalendar": [
        { "contract_number": "1", "settlement_date": "31.10.2026" }
      ],
      "combineCheck": [],
      "allocation": []
    }
  ]
}
```

- `"..."` ใน `conditionHeader` / `businessVolumeSales` = ค่าจาก input เดิม (`vendor_name`, `payment_term`, `header_text`, `set_of_field_combination`, `text_field_combination`, `selection_group`, `vendor` ฯลฯ)
- `businessVolumeSales` มี 5 แถว = สินค้า 2 ตัว × 2 Billing Type + สินค้า Exclusive 1 ตัว
- ถ้าเลือกครบ 18 สาขา (`KA_EXCEPT_ONLINE`) → สัญญาเดียวกันนี้ซ้ำ 7 ชุด (`contract_number` 1–7) ต่างกันที่ `sales_organization` ตาม [plant-sales-org.json](lookup/plant-sales-org.json)
- ถ้ามี Flow `DC`/`XD` → `settlement_option` = `1 - Settlement to Settlement Plant`, `settlement_plant` = `91KA`, `auto_allocation` = `X`, 1 สัญญาต่อ BBY
