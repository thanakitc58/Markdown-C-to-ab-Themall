# 02 — `BONUSBUYS[]`

> อ่าน [README.md](README.md) ก่อน (หลักการ + กฎ Autofill มาก่อน Lookup)
> ไฟล์ lookup ที่ใช้:
> - [lookup/purchasing-group.json](lookup/purchasing-group.json) — BU, Department
> - [lookup/mechanic.json](lookup/mechanic.json) — Legacy Promotion No.
> - [lookup/card-type.json](lookup/card-type.json) — `card[]`
> - [lookup/stores.json](lookup/stores.json) — 18 สาขา offline, รหัสพิเศษ, `material_or_group_type`

---

## 1. Autofill มาก่อน Lookup (เฉพาะไฟล์นี้)

| ค่า | ใช้ค่า autofill จาก input ก่อน | ถ้าว่าง → lookup |
|---|---|---|
| `bonusBuyHeader.department` | `CONDITIONS[].conditionHeader.department` | [purchasing-group.json](lookup/purchasing-group.json) → `department_code` (ค้นด้วย Pur Group) |
| `buy[]` / `get[]` `.ean` | `MATERIALS[].barcode` | – |
| ราคาทุน / ราคาขาย | ค่าใน `MATERIALS[]` (ดู README) | – |

ต้อง lookup เองเสมอ: BU ใน `description`, `legacy_promotion_number`, `card[]`, `stores`

---

## 2. `bonusBuyHeader`

ตอนนี้ convert ส่งมาแค่ `bonus_buy_number`, `bonus_buy_profile`, `mechanic`, `valid_time_from`, `valid_time_to`, `wbs_number`, `promotion_area`

| key | สถานะ | กฎ | Lookup | ตัวอย่าง #43 |
|---|---|---|---|---|
| `bonus_buy_profile` | มีแล้ว ✅ | `MATERIALS[].bonus_buy_profile` | – | `D001` |
| `mechanic` | มีแล้ว ✅ | `MATERIALS[].mechanic` | – | `2For` |
| `purchasing_group` | **เพิ่ม** | `MATERIALS[].purchasing_group` | – | `C03` |
| `promotion_number` | **เพิ่ม** | `MATERIALS[].number_of_promotion` | – | `1` |
| `department` | **เพิ่ม** | autofill `conditionHeader.department` ก่อน · ถ้าว่างใช้ Pur Group เปิด lookup | [purchasing-group.json](lookup/purchasing-group.json) | `DEP032` |
| `legacy_promotion_number` | **เพิ่ม** | Mechanic → Legacy (เทียบแบบ trim) | [mechanic.json](lookup/mechanic.json) | `2FOR` |
| `description` | **เพิ่ม** | สูตรข้อ 2.1 | [purchasing-group.json](lookup/purchasing-group.json) (`bu`) | `1_GNF_(C03)_M Price Super Shock V.1/2026_2For_30.09-12.11.26` |
| `promotion_area` | **แก้กฎ** | `P4` ถ้า plant ของ BBY มี `22KA` (online) · อื่นๆ `P1` | – | `P1` |
| `valid_time_from` / `valid_time_to` | **แก้กฎ** | `HEADER.timeFrom` / `timeTo` · ถ้าว่างใช้ `00:00:00` / `23:59:59` · ⚠️ Profile `P010` → **ว่าง** (macro ไม่ใส่เวลาให้ P010) | – | `00:00:00` / `23:59:59` |
| `wbs_number` | มีแล้ว ✅ | `HEADER.wbsNumber` (ว่างได้) | – | `AP.26.8883.10.CP.01` |
| `reference_code` | **เพิ่ม** | Profile `P010` + มี `promo_tag` → `QB` · อื่นๆ ว่าง | – | `""` |
| `online_description_en` / `online_description_th` | ว่าง | template Mer C ใหม่ไม่มีช่องนี้ | – | `""` |

> Chargeback ไม่อยู่ใน `bonusBuyHeader` — อยู่ที่ `HEADER.rebateChargeback` ซึ่งส่งต่อตามที่ user กรอก ([01-HEADER.md](01-HEADER.md))

### 2.1 สูตร `description`

```text
{prefix}{BBY No}_{BU}_({PurGroup})_{PromotionName}_{Mechanic}_{StartPart}-{EndDate dd.mm.yy}
```

| ส่วน | กฎ |
|---|---|
| `prefix` | ดูตาราง prefix ด้านล่าง |
| `BBY No` | `bonus_buy_number` |
| `BU` | Pur Group → `bu` ใน [purchasing-group.json](lookup/purchasing-group.json) (`GNF` / `GF` / `FF`) — lookup เองเสมอ |
| `PurGroup` | `MATERIALS[].purchasing_group` |
| `PromotionName` | `HEADER.promotionName` |
| `Mechanic` | `MATERIALS[].mechanic` ตามที่กรอก (เช่น `2For`) |
| `StartPart` | จาก `HEADER.periodFrom`: `dd` + `.mm` ถ้าเดือนต่างจากวันสิ้นสุด + `.yy` ถ้าปีต่างกัน |
| `EndDate` | `dd.mm.yy` ของ `HEADER.periodTo` |
| ล้างอักขระ | ลบ TAB, ขึ้นบรรทัด (`\n`), `"` และ `'` |
| ความยาว | ไม่เกิน 60 ตัวอักษร (POS ไม่เกิน 50) |

ตัวอย่าง `StartPart`:

| Start → End | ผล |
|---|---|
| 09.07.2026 → 22.07.2026 | `09-22.07.26` |
| 30.09.2026 → 12.11.2026 | `30.09-12.11.26` |
| 28.12.2026 → 05.01.2027 | `28.12.26-05.01.27` |

Prefix:

| เงื่อนไข | prefix |
|---|---|
| Contract Type ขึ้นต้น `Z2` | `CPS_` |
| Contract Type ขึ้นต้น `Z3` | `CPS3_` |
| Plant online อย่างเดียว (`22KA`) | `KA_` · มี Z2 ด้วย → `CPS_KA_` |
| BBY online ที่แยกออกมา (ข้อ 5) | `KA_` · มี Z2 ด้วย → `KA_CPS_` |
| Profile `P010` + มี `promo_tag` | `QB_` / `QB_CPS_` / `QB_CPS3_` |
| BBY ของ DC (ข้อ 4) | `DC_` |
| อื่นๆ | ไม่มี |

> ⚠️ แค่กรอกค่า Compensate แต่ไม่มี Contract Type `Z2` → **ไม่ใส่** `CPS_`

---

## 3. `BONUSBUYS[]` ส่วนอื่น

| key | ตอนนี้ | ต้องเป็น |
|---|---|---|
| `card[]` | `[]` | 19 รายการต่อ BBY ตาม [card-type.json](lookup/card-type.json): `{ "bonus_buy_number": "1", "card_type": "00001" }` … |
| `buy[]` / `get[]` `.material_or_group_type` | `"Material Group"` | BBY มี Material Grouping → `"MGPNew"` · ไม่มี → `"MAT"` (ใส่ทั้ง Buy และ Get) — ส่ง**ตัวย่อ** เพราะช่องนี้เก็บ short code ยาวไม่เกิน 6 ตัว ([stores.json](lookup/stores.json) → `material_or_group_type`) |
| `buy[]` / `get[]` `.ean` | EAN ของแถวแรก | autofill `MATERIALS[].barcode` · **ว่าง** เมื่อเป็น `MGPNew` |
| `buy[]` / `get[]` `.normal_cost_price_excluding_vat` | ราคาต่อลัง | `cost_price_case_excl_vat_normal / pack_size` (เช่น 1700.82 / 12 = 141.74) |
| `get[]` `.promotion_cost_price_excluding_vat` | ราคารวม VAT | `cost_price_unit_inc_vat_promo / 1.07` เมื่อ `sales_tax = V` |
| `stores[]` | 18 สาขา + `91KA` + `92KA` | ตามข้อ 3.1 |

### 3.1 กฎ `stores` (ตาม macro `GetPlant`)

SmartWeb ไม่มีช่อง "ALL Offline" → ดูจาก `MATERIALS[].stores` ของแถวแรกของ BBY · รายชื่อสาขาอยู่ใน [stores.json](lookup/stores.json)

| `MATERIALS[].stores` | ส่ง `stores` |
|---|---|
| มีครบ 18 สาขา offline (`offline_stores` ใน stores.json) | `["KA_EXCEPT_ONLINE"]` |
| มีแค่ `22KA` | `["22KA"]` (และ `promotion_area` = `P4`, prefix `KA_`) |
| มีทั้งสาขา offline และ `22KA` | BBY ปกติ = ส่วน offline ตามกฎด้านบน + สร้าง **BBY online แยกอีกชุด** ตามข้อ 5 |
| อื่นๆ | รายสาขาตามที่เลือก เช่น `["13KA", "14KA"]` |
| `91KA` / `92KA` | ตัดออกเสมอ (เป็น DC / FDC ไม่ใช่สาขาขาย — `91KA` ใช้เฉพาะ BBY DC ข้อ 4) |

---

## 4. แยก BBY ของ DC

ถ้ามีแถว `MATERIALS[].flow_type = DC` → สร้าง BBY **อีกชุด** สำหรับ DC (เฉพาะแถวที่ Flow = DC)

| | BBY ปกติ | BBY DC |
|---|---|---|
| `bonus_buy_number` | ตามแถว | เลขถัดไปต่อจาก BBY ชุดสุดท้าย (ห้ามซ้ำใน payload) |
| `stores` | ตามข้อ 3.1 | `["91KA"]` |
| `description` | ตามข้อ 2.1 | prefix `DC_` |
| `promotion_area` | ตามข้อ 2 | `P1` เสมอ |
| `card[]` | 19 รายการ | 19 รายการ |
| วันที่ | `HEADER.periodFrom` / `periodTo` | เหมือน BBY ปกติ (macro เลื่อนวันเริ่มเร็วขึ้น 7 วัน แต่ AB ไม่มีช่องวันที่ราย BBY และตกลงใช้ `HEADER` เดียวกัน) |
| สัญญา | ตาม Contract Type | ไม่สร้าง |

---

## 5. แยก BBY online

ถ้าแถวเลือกทั้งสาขา offline และ `22KA` → สร้าง BBY **อีกชุด** สำหรับ online (macro `Generate_Online_File`) เฉพาะสินค้าที่เปิดขาย online

| | BBY ปกติ (offline) | BBY online |
|---|---|---|
| `bonus_buy_number` | ตามแถว | เลขถัดไปต่อจาก BBY ชุดสุดท้าย (ห้ามซ้ำใน payload) |
| `stores` | ส่วน offline ตามข้อ 3.1 | `["22KA"]` |
| `description` | ตามข้อ 2.1 | prefix `KA_` (มี Z2 → `KA_CPS_`) |
| `promotion_area` | `P1` | `P4` |
| `card[]` | 19 รายการ | 19 รายการ |
| สัญญา (ถ้า Z2) | ตาม [03-CONDITIONS.md](03-CONDITIONS.md) | สร้างแยก ดู [03-CONDITIONS.md](03-CONDITIONS.md) ข้อ 9 |

BBY DC และ BBY online อยู่ใน payload เดียวกับ BBY ปกติ ใต้ `HEADER` ชุดเดียวกัน (ไม่แก้ `HEADER`)

---

## 6. ตัวอย่าง — SmartWeb #43 (ไม่มีสัญญา + มี DC)

Input:
- `HEADER`: Promotion Name `M Price Super Shock V.1/2026`, Period `30.09.2026`–`12.11.2026`, WBS `AP.26.8883.10.CP.01`
- `MATERIALS[]` แถวแรก: Pur Group `C03`, Profile `D001`, Mechanic `2For`, Vendor `BTG07`, Flow `DC`, Promotion No. `1`, เลือกครบ 18 สาขา offline, มี Material Grouping `MM8EB3EF31`
- `CONDITIONS[].conditionHeader` แถวแรก: Reason `1`, Contract Type ว่าง, Department `DEP032` (autofill)

Output:

```json
{
  "BONUSBUYS": [
    {
      "line_number": 1,
      "bonus_buy_number": "1",
      "bonusBuyHeader": {
        "bonus_buy_number": "1",
        "promotion_number": "1",
        "description": "1_GNF_(C03)_M Price Super Shock V.1/2026_2For_30.09-12.11.26",
        "purchasing_group": "C03",
        "bonus_buy_profile": "D001",
        "mechanic": "2For",
        "legacy_promotion_number": "2FOR",
        "department": "DEP032",
        "valid_time_from": "00:00:00",
        "valid_time_to": "23:59:59",
        "wbs_number": "AP.26.8883.10.CP.01",
        "promotion_area": "P1"
      },
      "stores": ["KA_EXCEPT_ONLINE"],
      "buy": [
        {
          "bonus_buy_number": "1",
          "material_or_group_type": "MGPNew",
          "material_or_group_code": "MM8EB3EF31",
          "ean": "",
          "normal_cost_price_excluding_vat": "141.74",
          "...": "..."
        }
      ],
      "get": [
        {
          "bonus_buy_number": "1",
          "material_or_group_type": "MGPNew",
          "material_or_group_code": "MM8EB3EF31",
          "ean": "",
          "normal_cost_price_excluding_vat": "141.74",
          "promotion_cost_price_excluding_vat": "141.74",
          "...": "..."
        }
      ],
      "card": [
        { "bonus_buy_number": "1", "card_type": "00001" },
        { "bonus_buy_number": "1", "card_type": "00002" },
        { "bonus_buy_number": "1", "card_type": "00003" },
        { "bonus_buy_number": "1", "card_type": "00004" },
        { "bonus_buy_number": "1", "card_type": "00005" },
        { "bonus_buy_number": "1", "card_type": "00006" },
        { "bonus_buy_number": "1", "card_type": "00007" },
        { "bonus_buy_number": "1", "card_type": "00008" },
        { "bonus_buy_number": "1", "card_type": "00009" },
        { "bonus_buy_number": "1", "card_type": "00010" },
        { "bonus_buy_number": "1", "card_type": "00011" },
        { "bonus_buy_number": "1", "card_type": "00012" },
        { "bonus_buy_number": "1", "card_type": "00013" },
        { "bonus_buy_number": "1", "card_type": "00014" },
        { "bonus_buy_number": "1", "card_type": "00015" },
        { "bonus_buy_number": "1", "card_type": "00016" },
        { "bonus_buy_number": "1", "card_type": "00017" },
        { "bonus_buy_number": "1", "card_type": "00018" },
        { "bonus_buy_number": "1", "card_type": "01045" }
      ],
      "...": "..."
    },
    {
      "line_number": 2,
      "bonus_buy_number": "2",
      "bonusBuyHeader": {
        "bonus_buy_number": "2",
        "promotion_number": "1",
        "description": "DC_2_GNF_(C03)_M Price Super Shock V.1/2026_2For_30.09-12.11.26",
        "purchasing_group": "C03",
        "bonus_buy_profile": "D001",
        "mechanic": "2For",
        "legacy_promotion_number": "2FOR",
        "department": "DEP032",
        "valid_time_from": "00:00:00",
        "valid_time_to": "23:59:59",
        "wbs_number": "AP.26.8883.10.CP.01",
        "promotion_area": "P1"
      },
      "stores": ["91KA"],
      "buy": [ { "bonus_buy_number": "2", "material_or_group_type": "MGPNew", "material_or_group_code": "MM8EB3EF31", "ean": "", "...": "..." } ],
      "get": [ { "bonus_buy_number": "2", "material_or_group_type": "MGPNew", "material_or_group_code": "MM8EB3EF31", "ean": "", "...": "..." } ],
      "card": [
        { "bonus_buy_number": "2", "card_type": "00001" },
        "... 19 รายการเหมือน BBY 1 แต่ bonus_buy_number = 2"
      ],
      "...": "..."
    }
  ]
}
```

- Reason `1` + ไม่มี Contract Type → ไม่มี prefix `CPS_`, `CONDITIONS` ว่าง
- Flow = DC → BBY ชุดที่ 2 (`DC_`, `91KA`) ใน payload เดียวกัน
- `"..."` = ช่องอื่นที่ convert ส่งอยู่แล้ว ไม่เปลี่ยน
