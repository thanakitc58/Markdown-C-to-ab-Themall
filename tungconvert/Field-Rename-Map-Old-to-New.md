# Field Rename Map — เก่า → ใหม่ (C / AB)

| | |
|--|--|
| **อัปเดต** | 2026-09-04 |
| **แหล่งใหม่** | `merCNewname.json` · `Abnewname.json` · `payload-keys.ts` · **`PROMOTION_COLUMN_DEFS.js`** |
| **แหล่งเก่า** | JSON ที่ convert/QA ใช้ตอนนี้ (`JsonBBY/*` · layout sample) |
| **ขอบเขต** | ฟิลด์ที่ **convert Mer C → AB ใช้จริง** ครอบทุก profile (D001/D002/D003/F001/F003/P001/P010/P011/P015) |

> **HEADER ส่วนใหญ่ชื่อเดิม** (camelCase ไม่เปลี่ยน) — ตารางด้านล่างเน้นที่เปลี่ยนจริง  
> Section โครง (`BONUSBUYS` / `CONDITIONS` / `MATERIALS` / `buy` / `get`) **ชื่อเดิม**

---

## สำคัญ: มีชื่อ 3 ชั้น (อย่าสับสน)

| ชั้น | ไฟล์ | ตัวอย่าง |
|------|------|----------|
| **A. JSON wire เก่า** (convert/QA ตอนนี้) | `JsonBBY/*` | `noof_promotion` · `field2` · `fieldP` · `cond_type_condition_rate` |
| **B. JSON wire ใหม่** (ทีมอื่น rename) | `merCNewname.json` · `Abnewname.json` | `number_of_promotion` · `material_or_group_type` · `promotion_price_thb` · `condition_rate` |
| **C. Grid column `value`** (UI) | **`PROMOTION_COLUMN_DEFS.js`** | `material_number_of_promotion` · `buy_material_or_group_type` · `get_promotion_price_thb` · `condition_type_condition_rate` |

`payload-keys.ts` → `PROMOTION_GRID_COLUMN` ยังชี้ไปชื่อสั้นชั้น A หลายจุด (เช่น `buy_f2`, `get_p`, `comp_set`) — **ยังไม่ตรง DEFS ทั้งหมด** ต้อง sync กัน

### จุดที่ DEFS ≠ ชื่อใน payload ใหม่ / เก่า (ต้องยืนยันกับทีม)

| ความหมาย | JSON เก่า (A) | JSON ใหม่ (B) | DEFS `value` (C) |
|-----------|---------------|---------------|------------------|
| MCH | `mch` | `mch` | `material_merchandise_hierarchy` |
| ชื่อ TH | `material_th_des` | `material_description_th` | **`material_th_des`** (ยังสั้น) |
| ชื่อ EN | `material_en_des` | `material_description_en` | **`material_en_des`** (ยังสั้น) |
| Vendor des | `vendor_des` | `vendor_description` | `material_vendor_description` |
| FOC | `cost_foc` | `cost_price_case_excl_vat_foc` | **`material_cost_free_of_charge`** |
| Cost promo unit | `cost_promo` | `cost_price_unit_inc_vat_promo` | `material_cost_promotion` |
| Compensate set | `comp_set` | `compensate_baht_per_set` | **`material_compensate_per_set`** |
| Promo tag | `promo_tag` | `promo_tag` | `material_promotion_tag` |
| Buy type | `field2` | `material_or_group_type` | `buy_material_or_group_type` |
| Buy code | `field4` | `material_or_group_code` | `buy_material_or_group_code` |
| Buy qty | `field9` | `minimum_quantity` | `buy_minimum_quantity` |
| Buy cost | `field7` | `normal_cost_price_excluding_vat` | **`buy_normal_cost_excluding_vat`** (ไม่มี `_price_`) |
| Get qty | `getQuantity` | `get_quantity` | **`get_get_quantity`** |
| Get ราคาโปร | `fieldP` | `promotion_price_thb` | `get_promotion_price_thb` |
| Get cost | `field7` | `normal_cost_price_excluding_vat` | `get_normal_cost_excluding_vat` |
| BBY header area | `promotionArea` | `promotion_area` | `header_promotion_area` |
| Cond reason | `cond_hdr_reason` | `reason` | `condition_header_reason` |
| Cond rate | `cond_type_condition_rate` | `condition_rate` | `condition_type_condition_rate` |
| Cond unit2 / scale | `cond_type_unit_2` | `scale_unit` (ใน payload-keys) | **`condition_type_unit_2`** (ยังชื่อ unit_2) |

> **สรุปจาก DEFS:** UI rename ไปทางชื่อยาวแล้ว แต่ยังเหลือชื่อสั้นปน (`material_th_des`, `condition_type_unit_2`) และบางชื่อคนละคำกับ JSON ใหม่ (`compensate_per_set` vs `compensate_baht_per_set`, `cost_free_of_charge` vs `cost_price_case_excl_vat_foc`)

---

## สรุปสั้นส่งทีมอื่น

| ชั้น | เปลี่ยนอะไร |
|------|-------------|
| HEADER | เกือบไม่เปลี่ยน |
| MATERIALS (C) | ชื่อสั้น → ชื่อยาว snake_case |
| BONUSBUYS header / buy / get (AB) | camelCase / `field*` → snake_case ความหมายชัด |
| CONDITIONS | `cond_hdr_*` / `cond_type_*` / `bv_*` → ชื่อสั้นใน object (ไม่มี prefix) |
| Meta | `lineNumber` → `line_number` · `status` → `STATUS` |

---

# 0) Root / Meta

| เก่า (ที่ใช้ตอนนี้) | ใหม่ | หมายเหตุ |
|---------------------|------|----------|
| `LAYOUT` | `LAYOUT` | เหมือนเดิม `"C"` / `"AB"` |
| `status` | `STATUS` | ขึ้นใหญ่ · ค่าเช่น `DRAFT` |
| `lineNumber` | `line_number` | ทุก array element |
| `contractNumber` (บน CONDITIONS item) | `contract_number` | |
| `MATERIALS` | `MATERIALS` | เหมือนเดิม |
| *(ไม่มี / ว่าง)* | `MATERIALGROUPINGS` | array ใหม่ฝั่ง C/AB |
| `BONUSBUYS` / `CONDITIONS` | เหมือนเดิม | |

---

# 1) HEADER — ส่วนใหญ่ไม่เปลี่ยน

ใช้ร่วมทุก profile · **ชื่อเดิม** ตามตัวอย่างใหม่:

`theme` · `promotionName` · `group` · `purchasingGroup` · `contractType` · `bonusBuyProfile` · `rebateChargeback` · `singleMultiple` · `wbsNumber` · `volume` · `vendorCode` · `vendorName` · `periodFrom` · `periodTo` · `timeFrom` · `timeTo` · `days`

| เก่า | ใหม่ | หมายเหตุ |
|------|------|----------|
| `createdBy` / `updatedBy` | อาจมีทั้ง camel + snake (`created_by`) | ตัวอย่างใหม่มีคู่ซ้ำ — ทีมยืนยัน key เดียว |
| `status` ใน HEADER | อาจย้ายไป root `STATUS` | |

**Convert ใช้จาก HEADER:** `wbsNumber` · `timeFrom`/`timeTo` · `rebateChargeback` · `vendorCode` (→ NOBP) · `period*` · `days` · `singleMultiple`

---

# 2) MATERIALS (Layout C) — สำคัญสุดต่อ convert

| เก่า (QA / convert ตอนนี้) | ใหม่ (`merCNewname` / `payload-keys`) | ใช้ทำอะไร |
|----------------------------|----------------------------------------|-----------|
| `noof_promotion` | `number_of_promotion` | group BBY |
| `noof_material_grouping` | `number_of_material_grouping` | group BBY |
| `noof_bonus_buy` | `number_of_bonus_buy` | `bonusBuyNumber` |
| `pur_group` | `purchasing_group` | header/copy |
| `material` | `material` | buy/get code |
| `barcode` | `barcode` | ean |
| `material_th_des` / `material_des` | `material_description_th` | description |
| `material_en_des` / `material_des_2` | `material_description_en` | sap master |
| `vendor` | `vendor` | CONDITIONS vendor |
| `vendor_des` | `vendor_description` | vendor name |
| `flow_type` | `flow_type` | |
| `pack_size` | `pack_size` | |
| `sales_unit` | `sales_unit` | unit |
| `sales_tax` | `sales_tax` | |
| `fix_gp` | `fix_gross_profit` | |
| `cost_normal` | `cost_price_case_excl_vat_normal` | |
| `cost_discount_amt` | `cost_price_case_excl_vat_discount_amount` | |
| `cost_discount` | `cost_price_case_excl_vat_discount_percent` | |
| `cost_foc` | `cost_price_case_excl_vat_foc` | FOC (optional) |
| `cost_date_start` | `cost_price_case_excl_vat_date_start` | FOC dates |
| `cost_date_end` | `cost_price_case_excl_vat_date_end` | |
| `cost_normal_2` | `cost_price_unit_inc_vat_normal` | buy/get cost |
| `cost_promo` | `cost_price_unit_inc_vat_promo` | |
| `sales_price_normal` | `sales_price_normal` | field8 |
| `sales_price_promo` | `sales_price_promotion` | **fieldP** |
| `bonus_buy_profile` | `bonus_buy_profile` | router profile |
| `mechanic` | `mechanic` | router mechanic |
| `discount_amt` | `discount_amount` | [R] |
| `disc_for_p015` | `discount_percent_for_p015` | P015 |
| `disc_deal` | `discount_percent_deal` | |
| `disc_plu` | `discount_percent_plu` | [%] |
| `comp_qty_in_sap` | `compensate_quantity_in_sap` | **condition_rate** |
| `comp_set` | `compensate_baht_per_set` | rate fallback |
| `comp_f3` | `compensate_percent` | rate % |
| `gp_normal` | `gross_profit_normal` | |
| `gp_promo` | `gross_profit_promotion` | |
| `mix` / `media` / `promo_tag` / `remind` / `remarks` | ชื่อเดิม snake ยาวขึ้นตาม payload-keys | |
| `forecast_qty` | `forecast_quantity` | |
| `forecast_amt` | `forecast_amount` | |
| `sqm_jun` | `sales_quantity_1_month_ago` | |
| `sqm_may` | `sales_quantity_2_months_ago` | |
| `sqm_apr` | `sales_quantity_3_months_ago` | |
| `avg_sales_qty_day` | `average_sales_quantity_per_day` | |
| `disp_mprice_category` | `display_promotion_m_price_category` | **ห้าม** map เป็น promotionArea |
| `disp_mprice_code` | `display_promotion_m_price_code` | |
| `stores` | `stores` | |
| `forecastStore` / `planogramStore` | เหมือนเดิม · item อาจมีทั้ง `storeCode` + `store_code` | |

### Alias เก่าที่เคยเจอ (camelCase layout-c)

| เก่าอีกแบบ | ใหม่ |
|------------|------|
| `numberOfPromotion` | `number_of_promotion` |
| `numberOfMaterialGrouping` | `number_of_material_grouping` |
| `numberOfBonusBuy` | `number_of_bonus_buy` |
| `purchasingGroup` | `purchasing_group` |
| `materialDescription` | `material_description_th` |
| `materialDescription2` | `material_description_en` |
| `vendorDescription` | `vendor_description` |
| `salesPricePromotion` | `sales_price_promotion` |
| `bonusBuyProfile` (บนแถว) | `bonus_buy_profile` |

---

# 3) BONUSBUYS (Layout AB)

## 3.1 ก้อน BBY

| เก่า | ใหม่ |
|------|------|
| `lineNumber` | `line_number` |
| `bonusBuyNumber` | `bonus_buy_number` |

## 3.2 `bonusBuyHeader` — ใช้ทุก profile

| เก่า (camelCase) | ใหม่ (snake_case) | หมายเหตุ |
|------------------|-------------------|----------|
| `bonusBuyNumber` | `bonus_buy_number` | |
| `promotionNumber` | `promotion_number` | |
| `description` | `description` | |
| `purchasingGroup` | `purchasing_group` | |
| `bonusBuyProfile` | `bonus_buy_profile` | D001/P010/… |
| `mechanic` | `mechanic` | |
| `limitNumber` | `limit_number` | |
| `validTimeFrom` | `valid_time_from` | |
| `validTimeTo` | `valid_time_to` | |
| `product` | `product` | |
| `priceTag` | `price_tag` | |
| **`promotionArea`** | **`promotion_area`** | ค่า `P1`/`P4` |
| `wbsNumber` | `wbs_number` | |
| `referenceBonusBuy` | `reference_bonus_buy` | |
| `legacyPromotionNumber` | `legacy_promotion_number` | |
| `allowDiscountAfterGetExtraMPoint` | `allow_discount_after_get_extra_m_point` | |
| `notAcceptAnyDiscountCoupon` | `not_accept_any_discount_coupon` | |
| `notAcceptAnyDiscountCard` | `not_accept_any_discount_card` | |
| `notAllowForEmployee` | `not_allow_for_employee` | |
| `referenceCode` | `reference_code` | P010 |
| `department` | `department` | |
| `onlineDescriptionEnglish` | `online_description_en` | P4 |
| `onlineDescriptionThai` | `online_description_th` | P4 |

## 3.3 `buy[]` — D001/D002/F001/D003(A) ฯลฯ

| เก่า | ใหม่ | ความหมาย |
|------|------|----------|
| `bonusBuyNumber` | `bonus_buy_number` | |
| **`field2`** | **`material_or_group_type`** | `"Material"` → มักเป็น `"MAT"` / `"MCH"` |
| **`field4`** | **`material_or_group_code`** | รหัสสินค้า/กลุ่ม |
| `description` | `description` | |
| `sapMasterDescription` | `sap_master_description` | |
| **`field7`** | **`normal_cost_price_excluding_vat`** | cost |
| **`field8`** | **`normal_sales_price`** | ราคาปกติ |
| **`field9`** | **`minimum_quantity`** | qty buy |
| **`field10`** | **`minimum_amount`** | amount buy (บาง profile) |
| `ean` | `ean` | |
| `serial` | `serial` | |
| `salesUnit` | `sales_unit` | |
| `promotionTagSizeA4Cut*` | `pro_tag_size_a4_cut_*` | |

> `char` (เก่า) — ในชุดใหม่ตัวอย่างไม่มี · เช็คกับทีม schema

## 3.4 `get[]` — ทุก profile ที่มี get

| เก่า | ใหม่ | ความหมาย |
|------|------|----------|
| `bonusBuyNumber` | `bonus_buy_number` | |
| **`field2`** | **`material_or_group_type`** | |
| **`field4`** | **`material_or_group_code`** | |
| **`field5`** | **`material_or_group_description`** | |
| `sapMasterDescription` | `sap_master_description` | |
| **`field7`** | **`normal_cost_price_excluding_vat`** | |
| **`field8`** | **`normal_sales_price`** | |
| `getQuantity` | `get_quantity` | |
| `tierNumber` | `tier_number` | |
| **`field17`** | **`promotion_cost_price_excluding_vat`** | |
| **`fieldP`** | **`promotion_price_thb`** | ราคาโปร ⭐ |
| **`fieldR`** | **`discount_amount_thb`** | ส่วนลดบาท |
| **`field22`** / `%` | **`discount_percent`** | ส่วนลด % |
| `ean` | `ean` | |
| `unit` | `unit` | |
| `serialNumber` | `serial_number` | |
| `vat` / `vat2` | `vat_normal_price` / `vat_promotion_price` | |
| `grossProfit` / `grossProfit2` | `gp_normal_price` / `gp_promotion_price` | |
| `newGrossProfitRate` | `new_gross_profit_rate` | |
| `priceUnit` | `price_unit` | |
| `unitOfMeasure` | `unit_of_measure` | |
| `basicPoint` / `extraPoint` / `pointAmount` | `basic_point` / `extra_point` / `point_amount` | |
| `exclusion` / `noDiscount` | `exclusion` / `no_discount` | |
| `recursive` / `progressive` | `recursive` / `progressive` | |
| `tierQuantityA` / `tierAmountB` | `tier_quantity_a` / `tier_amount_b` | |

## 3.5 arrays อื่นใน BBY (ชื่อ section เดิม · key ข้างในเป็น snake)

| Section | ตัวอย่างเก่า → ใหม่ |
|---------|---------------------|
| `card` | `cardType` → `card_type` |
| `tender` | `tenderType` → `tender_type` |
| `installment` | `interestRate` → `interest_rate` · `numberOfMonths` → `number_of_months` |
| `posTerminal` | `terminalNumber` → `terminal_number` |
| `premium` | `stockPlant` → `stock_plant` · `amountForRedeem` → `amount_for_redeem` |
| `limitControl` | `perBonusBuy` → `per_bonus_buy` · `limitNumber` → `limit_number` |
| `coupon` | `logo1` → `logo_1` (header/detail/trailer คล้ายเดิม) |

---

# 4) CONDITIONS — prefix หาย · เหลือชื่อธุรกิจ

โครงเดิม: `conditionHeader` / `businessVolumePurchase` / `businessVolumeSales` / `conditionType`  
**ข้างใน:** จาก `cond_hdr_*` → ชื่อสั้น

## 4.1 `conditionHeader`

| เก่า (ใน JSON AB ที่ใช้) | ใหม่ |
|--------------------------|------|
| `cond_hdr_reason` | `reason` |
| `cond_hdr_contract_type` | `contract_type` |
| `cond_hdr_vendor` | `vendor` |
| `cond_hdr_vendor_name` | `vendor_name` |
| `cond_hdr_department` | `department_mer` |
| `cond_hdr_start` | `start` |
| `cond_hdr_end` | `end` |
| `cond_hdr_payment_term` | `payment_term` |
| `cond_hdr_payment_method` | `payment_method` |
| `cond_hdr_sales_organization` | `sales_organization` |
| `cond_hdr_settlement_option` | `settlement_option` |
| `cond_hdr_settlement_plant` | `settlement_plant` |
| `cond_hdr_header_text` | `header_text` |
| `cond_hdr_additional_text` | `additional_text` |
| `cond_hdr_department_2` | `department` |
| `cond_hdr_auto_allocation` | `auto_allocation` |
| `cond_hdr_*` อื่น | ตัด prefix `cond_hdr_` แล้วเป็น snake_case ตาม `payload-keys.ts` |

## 4.2 `conditionType` — rate ที่เทสCompensate

| เก่า | ใหม่ |
|------|------|
| `cond_type_condition_table` | `condition_table` |
| `cond_type_condition_type` | `condition_type` |
| `cond_type_brand_name` | `brand_name` |
| `cond_type_valid_from` | `valid_from` |
| `cond_type_valid_to` | `valid_to` |
| **`cond_type_condition_rate`** | **`condition_rate`** |
| `cond_type_condition_unit` | `condition_unit` |
| `cond_type_unit_2` | `scale_unit` |
| `cond_type_contract_no` | `contract_number` |
| `cond_type_per` | `per` |
| `cond_type_unit` | `unit` |

## 4.3 `businessVolumePurchase` / `businessVolumeSales`

| เก่า | ใหม่ |
|------|------|
| `bv_pur_field_combination` | `field_combination` |
| `bv_pur_text_field_combination` | `text_field_combination` |
| `bv_pur_contract_no` | `contract_number` |
| `bv_pur_set_of_field_combination` | `set_of_field_combination` |
| `bv_sales_inclusive_exclusive` | `inclusive_exclusive` |
| `bv_sales_vendor` | `vendor` |
| `bv_sales_vendor_name` | `vendor_name` |
| `bv_sales_billing_type` | `billing_type` |
| `bv_sales_contract_no` | `contract_number` |
| `bv_sales_bonus_buy` | `bonus_buy` |
| … | ตัด `bv_pur_` / `bv_sales_` แล้ว snake_case |

---

# 5) คีย์ที่ convert ต้องอ่าน/เขียน — checklist ทุก profile

### อ่านจาก C MATERIALS
`number_of_promotion` · `number_of_material_grouping` · `number_of_bonus_buy` · `material` · `barcode` · `material_description_*` · `vendor` · `sales_unit` · `sales_price_normal` · `sales_price_promotion` · `bonus_buy_profile` · `mechanic` · `discount_*` · `compensate_*` · `stores` · cost fields ตาม profile

### เขียน AB BONUSBUYS
`bonus_buy_number` · `bonus_buy_profile` · `mechanic` · `promotion_area` · `valid_time_*` · `wbs_number` · buy/get: `material_or_group_type` · `material_or_group_code` · `minimum_quantity` · `get_quantity` · `promotion_price_thb` / `discount_amount_thb` / `discount_percent`

### เขียน AB CONDITIONS
`reason` · `vendor` · **`condition_rate`** · `condition_table` · dates · bv fields

---

# 6) ตัวอย่างเทียบสั้นๆ

### Group keys (C)

```json
// เก่า
{ "noof_promotion": 1, "noof_material_grouping": 2, "noof_bonus_buy": 1, "comp_qty_in_sap": 0.5 }

// ใหม่
{ "number_of_promotion": 1, "number_of_material_grouping": 2, "number_of_bonus_buy": 1, "compensate_quantity_in_sap": 0.5 }
```

### Buy / Get (AB)

```json
// เก่า
{ "field2": "Material", "field4": "1000728050", "field9": 2, "fieldP": "555", "getQuantity": 2 }

// ใหม่
{
  "material_or_group_type": "MAT",
  "material_or_group_code": "1000728050",
  "minimum_quantity": "2",
  "promotion_price_thb": "555",
  "get_quantity": "2"
}
```

### Condition rate (AB)

```json
// เก่า
{ "cond_type_condition_rate": 0.5 }

// ใหม่
{ "condition_rate": 0.5 }
```

---

# 7) หมายเหตุให้ทีม rename

1. **ค่า business ไม่เปลี่ยน** — เปลี่ยนแค่ชื่อ key (`P1` ยังเป็น `P1`, mechanic ยังเป็น `2For`)
2. **`field2` = `"Material"`** ฝั่งใหม่อาจเป็น **`"MAT"`** — ยืนยัน enum กับ backend
3. รองรับ **alias ชั่วคราว** (อ่านทั้งเก่า+ใหม่) ช่วง transition จะปลอดภัยสุด
4. Source map ใน repo: `payload-keys.ts` → `PROMOTION_GRID_COLUMN` (new → old short) และไฟล์ตัวอย่าง `merCNewname.json` / `Abnewname.json`

---

## ไฟล์อ้างอิง

| ไฟล์ | บทบาท |
|------|--------|
| `merCNewname.json` | ตัวอย่าง Layout C ชื่อใหม่ (JSON wire) |
| `Abnewname.json` | ตัวอย่าง Layout AB ชื่อใหม่ (JSON wire) |
| `payload-keys.ts` | whitelist + map new ↔ grid เก่าสั้น |
| `payload-keys-old.ts` | ชุด camelCase เก่า (wire เดิม) |
| **`PROMOTION_COLUMN_DEFS.js`** | Grid UI `value` 323 คอลัมน์ (ชั้น C) — ชื่อจริงบนจอ/export |
