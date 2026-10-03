# Mer C convert → AB — ของที่ทำแล้ว / ของที่ยังไม่ตามมาโคร

อ้างอิงมาโคร: `Md/Mer-C_Convert_To_STD.txt`  
โค้ดหลัก: `constants/promotion-payload.js` → `convertLayoutCToAbc`  
เทสต์: `tests/merc-ab-bby-source.test.js`, `tests/promotion-layout-c-material-grouping.test.js`

ขอบเขตฝั่ง convert: **JSON ตอนเซฟ Layout C** ไม่รวม mapper Excel / กริด Vue / modal สาขา (ยกเว้นที่ระบุ)

---

## ของที่เพิ่มไปแล้ว (ตามมาโคร)

### 1. HEADER

- ส่งต่อตามที่ user กรอก ห้ามทับ / ห้าม remap
- ค่าที่ต้องตัดรหัสสั้น ใช้แค่ตอนสร้าง BBY / CONDITIONS ภายใน

### 2. bonusBuyHeader

- อ่านจากแถวแรกของแต่ละ BBY + lookup (`readMerCBbySource`)
- Pur Group, department, legacy promotion number
- `promotion_area`: มี `22KA` → `P4` นอกนั้น `P1`
- P010 + promo tag → `reference_code = "QB"`
- description ตามสูตรมาโคร (prefix + เลข BBY + BU + Pur Group + ชื่อโปร + mechanic + ช่วงวัน)
- ตัดความยาว 60 ตัว (POS ตัด 50)
- P010 ส่ง `valid_time_from` / `valid_time_to` เป็น `""`

Prefix ที่รองรับแล้ว:

- ปกติ Z2 → `CPS_` · Z3 → `CPS3_`
- DC → `DC_` / `DC_CPS_` / `DC_CPS3_`
- online แยกก้อน → `KA_` / `KA_CPS_`
- ออนไลน์อย่างเดียว → `KA_` / `CPS_KA_`
- P010 มี tag → `QB_` / `QB_CPS_` / `QB_CPS3_`

### 3. Stores (`GetPlant`)

ไฟล์: `promotion-payload.js` — `collapseOfflineStores`, `finalizeMerCPickerStores`, `checkedStores`, `resolveMerCStores`

| เคส | ผล |
|---|---|
| Layout C กด All / All Store | `["KA_EXCEPT_ONLINE"]` |
| เลือกครบสาขาขายที่มีใน picker (ตัด 91KA/92KA/22KA) | `["KA_EXCEPT_ONLINE"]` |
| ครบ 18 offline จาก `stores-lookup.json` | `["KA_EXCEPT_ONLINE"]` |
| มี 22KA ด้วย | `["KA_EXCEPT_ONLINE", "22KA"]` แล้วแยกก้อน online |
| ไม่ครบ | ส่งรายสาขา |
| 91KA / 92KA | ตัดออกจากก้อนขาย · ก้อน DC ใช้ `["91KA"]` |

หมายเหตุ UI: modal ดึงจาก `master_plant_bu` ชุดปัจจุบันมักไม่มี `35KA` / `68KA` และมี DC ปน — convert เลยถือว่า “ติ๊กครบใน picker” = All ไม่บังคับครบ 18 ตัวใน lookup

### 4. แยกก้อน BBY

- ก้อนปกติ
- แถว Flow = DC → ก้อน `stores: ["91KA"]` เลขต่อท้าย
- มีทั้งร้าน + 22KA → ก้อน `stores: ["22KA"]` area `P4`
- ลำดับ: ต่อก้อนวาง DC แล้วตามด้วย online ของก้อนนั้น
- มี `number_of_bonus_buy` ใช้เลขนั้น ไม่ยุบก้อน

### 5. Card type

- JSON `BONUSBUYS[].card` = 19 รายการจาก `constants/card-type-lookup.json`
- ยังไม่ขึ้นกริด Layout C และ mapper Excel ยังไม่เขียนชีต Card (ดูส่วนค้าง)

### 6. Buy / Get (profile ที่รองรับ)

รองรับแล้ว: `D001` `D002` `D003` `P001` `P010` `P011` `P015` `F001` `F003`

- มี `number_of_material_grouping` → type `MGPNew` + ชื่อกลุ่ม
- ไม่มี → type `MAT` + รหัสสินค้า
- MGPNew ไม่ใส่ EAN
- ทุนหาร pack size และหาร 1.07 เมื่อภาษี `V`
- qty จาก `mechanic-lookup.json` ไม่เจอ = 1
- ข้ามแถว Reject / convert แล้ว / mechanic On Pack

### 7. CONDITIONS / contract no (`Fill_Contract`)

`buildMerCConditions`

- สร้างเมื่อ contract type ขึ้นต้น `Z2` เท่านั้น
- ก้อน DC ไม่สร้างสัญญา
- เลขเริ่ม `"1"` บวกตาม Sales Org
- Option 1 → settlement plant (DC/XD → `91KA` + `auto_allocation = X`)
- Option 2 + All → 7 Sales Org
- Online → option 2, Sales Org 3010
- `conditionType` ตามตาราง V 163 รายสินค้า
- `businessVolumeSales` = สินค้า × ZFP0/ZFP2 + Exclusive
- ไม่มี Z2 → `CONDITIONS: []` (ถูกแล้ว)

อย่าใส่ `renameConditionNode` — เป็นร่างใน editor ไม่ได้อยู่ในไฟล์เซฟ และไม่ได้สร้างเลขสัญญา

### 8. MATERIALGROUPINGS

- 1 แถวต่อ MATERIALS ที่มีเลข grouping
- `running_number` ← `number_of_material_grouping`
- `category` = `1 - Material No.`
- `component` = รหัสสินค้า

---

## บั๊กที่ต้องแก้ก่อน — ชื่อกลุ่มไม่ตรง Buy/Get

**อาการ:** ชีต Material Grouping เป็นชื่อหนึ่ง แต่ Buy/Get เป็นอีกชื่อ หรือยังเป็นรหัสสินค้า  
ตัวอย่างที่เจอ: `MM8F081441` vs `MM8EF10BD1`

**สาเหตุ:** ชื่อกลุ่มมี 2 แหล่ง

1. Buy/Get ใช้ชื่อที่ `ensureGroupingNames` gen ใหม่ทุกครั้ง (`MM` + hex วินาที + running)
2. `MATERIALGROUPINGS` ถ้ามีแท็บ grouping (`subTables[1]`) จะใช้ `row.c2` จาก UI ไม่ใช้ map ชุดเดียวกับข้อ 1

จุดโค้ด: `convertLayoutCToAbc` ประมาณบรรทัด 2886–2901 ใน `promotion-payload.js`

มาโครทำทางเดียว: เขียนเลข grouping ลงชีต Material Grouping คอลัมน์ A → Excel gen ชื่อคอลัมน์ B → เอาชื่อนั้นไปใส่ Buy/Get

**วิธีแก้ที่ถูก:** เหลือ map เดียว

1. เติม `groupingNameByRunning` จากชื่อที่มีอยู่ก่อน (`subTables[1].c2` หรือ `MATERIALGROUPINGS.grouping_name`) ตาม `running_number`
2. running ที่ยังไม่มีชื่อ ค่อย gen `MMxxxx` ครั้งเดียว
3. ทั้ง `MATERIALGROUPINGS` และ `buy[]` / `get[]` อ่านจาก map นี้เท่านั้น

ไม่ต้องลบสูตร gen ทิ้ง (ยังใช้ตอนยังไม่มีชื่อ)  
ไม่ต้องลบ `subTables` ทิ้ง (ถ้า UI มีชื่อแล้วให้ใช้ชื่อนั้น)

อย่าไปแก้ mapper Excel ก่อน จนกว่า JSON `BONUSBUYS.buy/get.material_or_group_code` จะเท่ากับ `MATERIALGROUPINGS.grouping_name` ของ running เดียวกัน

---

## ของที่ยังไม่ตามมาโคร

| เรื่อง | มาโคร | Convert ตอนนี้ | ใครควรทำ |
|---|---|---|---|
| สูตร buy/get ราย `Gen_*` ของ 9 profile ที่รองรับ | แต่ละ profile มีสูตรราคา / % / qty เอง | ใช้ builder รวม ยังไม่ไล่ครบสูตรในไฟล์มาโคร | convert (อย่าเดา ต้องเปิด `Gen_*` ในมาโคร) |
| FOC / MEK1 | มีใน flow เดิม | ยังไม่มี | convert |
| Card ขึ้นกริด / Excel | `Fill_Card_Type` เขียนชีต Card | มีใน JSON แล้ว mapper/กริดยังไม่เขียน | Excel mapper + กริด AB |
| แถว Material Grouping เป็น `0` / `FALSE` | เทมเพลต padding | ไม่เกี่ยวกับ convert | ทีม Excel |
| Modal ไม่มี 35KA / 68KA | มาโครมี 18 สาขาขาย | convert ยุบ All จาก picker ได้แล้ว | master_plant_bu / UI ถ้าจะให้ครบ 18 จริง |

`renameConditionNode` / `renameKeysByPrefix` **ไม่ต้องเพิ่ม** — convert เขียน `contract_number` อยู่แล้ว

---

## ไฟล์ที่เกี่ยวกับงานนี้

- `constants/promotion-payload.js` — convert ทั้งหมด
- `constants/stores-lookup.json` — 18 สาขา offline
- `constants/card-type-lookup.json` — 19 card
- `constants/plant-sales-org-lookup.json` — แตกสัญญา option 2
- `constants/mechanic-lookup.json` — buy/get qty
- `constants/mechanic-convert-lookup.json` — legacy / qty_set
- `Md/Mer-C_Convert_To_STD.txt` — มาโครต้นทาง

---

## วิธีเช็คหลังแก้ชื่อกลุ่ม

1. Layout C กรอก `number_of_material_grouping` (เช่น 1, 1, 2)
2. Save as Draft
3. ดู console `🚀 [Converted Layout C -> AB]`
4. `MATERIALGROUPINGS[].grouping_name` ของ running เดียวกัน ต้องเท่ากับ `BONUSBUYS[].buy/get[].material_or_group_code`
5. type ต้องเป็น `MGPNew` ไม่ใช่รหัสสินค้า
6. รัน `npm test -- --testPathPattern='merc-ab-bby-source|promotion-layout-c-material-grouping'`

ถ้า JSON ตรงแล้ว แต่ Excel ยังไม่ขึ้น → ส่งต่อทีม mapper ไม่ใช่ convert
