# P010 / Price Off / Promo baseline

## ดูซ้าย-ขวาเต็มจอ (แนะนำ)

เปิดไฟล์ HTML นี้:

- [`PriceOff-Promo-baseline.html`](./PriceOff-Promo-baseline.html)
- Preview เต็มจอ: [htmlpreview](https://htmlpreview.github.io/?https://github.com/thanakitc58/Markdown-C-to-ab-Themall/blob/main/qa-evidence/P010/PriceOff-Promo-baseline.html)

> หน้า GitHub `.md` บีบความกว้างตาราง ทำให้ต้องเลื่อนซ้ายขวา จึงย้ายการเทียบแบบเต็มจอไปที่ HTML

## ไฟล์ดิบ

- [AB JSON](./PriceOff-Promo-baseline.ab.json)
- [C JSON](./PriceOff-Promo-baseline.c.json)

## อ่านใน GitHub แบบเต็มความกว้าง (เรียงบน-ล่าง)

### AB JSON

```json
{
  "LAYOUT": "AB",
  "HEADER": {
    "group": "MERCHANDISE SUPERMARKET(GROCERY FOOD)",
    "promotionName": "M Price 09/2026",
    "purchasingGroup": "C02",
    "theme": "C011",
    "bonusBuyProfile": "D002",
    "rebateChargeback": "",
    "contractType": "",
    "wbsNumber": "AP.26.8883.10.CP.01",
    "vendorCode": "NOBP",
    "vendorName": "",
    "periodFrom": "2026-09-19",
    "periodTo": "2026-09-29",
    "days": [
      "All"
    ],
    "singleMultiple": "Multiple",
    "volume": "",
    "timeFrom": "",
    "timeTo": ""
  },
  "BONUSBUYS": [
    {
      "line_number": 1,
      "bonus_buy_number": "1",
      "bonusBuyHeader": {
        "bonus_buy_number": "1",
        "bonus_buy_profile": "P010",
        "mechanic": "Price Off",
        "wbs_number": "AP.26.8883.10.CP.01",
        "reference_code": "",
        "promotion_area": "P1"
      },
      "buy": [],
      "get": [
        {
          "bonus_buy_number": "1",
          "material_or_group_type": "Material Group",
          "material_or_group_code": "MM8D1C22C1",
          "get_quantity": 1,
          "material_or_group_description": "แคนเมค มาร์ชเมลโลว์พาวเดอร์อะบลูม#01",
          "sap_master_description": "CANMAKE MARSHMALLOW POWDER ABLOOM #01",
          "normal_cost_price_excluding_vat": "329.72",
          "normal_sales_price": "490",
          "promotion_cost_price_excluding_vat": "352.8",
          "promotion_price_thb": "500",
          "ean": "4901008314037",
          "unit": "EA"
        }
      ],
      "stores": [
        "13KA",
        "14KA",
        "15KA",
        "16KA",
        "17KA",
        "30KA",
        "31KA",
        "32KA",
        "33KA",
        "34KA",
        "60KA",
        "61KA",
        "64KA",
        "65KA",
        "66KA",
        "67KA",
        "91KA",
        "92KA"
      ],
      "card": [],
      "tender": [],
      "installment": [],
      "posTerminal": [],
      "premium": [],
      "coupon": [],
      "limitControl": []
    }
  ],
  "CONDITIONS": [
    {
      "line_number": 0,
      "contract_number": "1",
      "conditionHeader": {
        "reason": "X",
        "contract_type": "Z200",
        "vendor": "MSH02",
        "vendor_name": "บริษัท เอ็ม. เอส. ฮานาโซโน",
        "department_mer": "Mer C",
        "start": "19.09.2026",
        "end": "29.09.2026",
        "payment_method": "M",
        "sales_organization": "2009",
        "settlement_option": "1",
        "header_text": "M-Price",
        "additional_text": "M-Price",
        "department": "DEP033"
      },
      "businessVolumePurchase": [
        {
          "set_of_field_combination": "BVCU",
          "field_combination": "Z256",
          "text_field_combination": "Vendor, Material, Cond. Type, Sales Unit",
          "inclusive_exclusive": "Inclusive",
          "contract_number": "1"
        }
      ],
      "businessVolumeSales": [
        {
          "set_of_field_combination": "BVCU",
          "field_combination": "Z256",
          "text_field_combination": "Vendor, Material, Cond. Type, Sales Unit",
          "inclusive_exclusive": "Inclusive",
          "selection_group": "Z001",
          "vendor": "MSH02",
          "vendor_name": "บริษัท เอ็ม. เอส. ฮานาโซโน",
          "bonus_buy": "1",
          "contract_number": "1"
        }
      ],
      "conditionType": [
        {
          "condition_table": "V 163",
          "condition_type": "1",
          "brand_name": "บริษัท เอ็ม. เอส. ฮานาโซโน",
          "valid_from": "19.09.2026",
          "valid_to": "29.09.2026",
          "condition_rate": 1,
          "condition_unit": "1",
          "scale_unit": "EA",
          "contract_number": "1"
        }
      ],
      "settlementCalendar": [],
      "combineCheck": [],
      "allocation": []
    }
  ],
  "MATERIALS": [
    {
      "number_of_promotion": 1,
      "number_of_material_grouping": 1,
      "mch": "HBA",
      "purchasing_group": "C05",
      "material": "1000728050",
      "barcode": "4901008314037",
      "material_description_th": "แคนเมค มาร์ชเมลโลว์พาวเดอร์อะบลูม#01",
      "material_description_en": "CANMAKE MARSHMALLOW POWDER ABLOOM #01",
      "vendor": "MSH02",
      "vendor_description": "บริษัท เอ็ม. เอส. ฮานาโซโน",
      "flow_type": "DS",
      "pack_size": 1,
      "sales_unit": "EA",
      "sales_tax": "V",
      "cost_price_case_excl_vat_normal": 329.72,
      "cost_price_unit_inc_vat_normal": 352.8,
      "cost_price_unit_inc_vat_promo": "352.8",
      "sales_price_normal": 490,
      "sales_price_promotion": 500,
      "bonus_buy_profile": "P010",
      "mechanic": "Price Off",
      "discount_percent_deal": "2.04%",
      "gross_profit_normal": "28%",
      "gross_profit_promotion": "29.44%",
      "forecast_quantity": "-",
      "forecast_amount": "-",
      "sales_quantity_1_month_ago": 8,
      "sales_quantity_2_months_ago": 4,
      "sales_quantity_3_months_ago": 7,
      "stores": [
        "13KA",
        "14KA",
        "15KA",
        "16KA",
        "17KA",
        "30KA",
        "31KA",
        "32KA",
        "33KA",
        "34KA",
        "60KA",
        "61KA",
        "64KA",
        "65KA",
        "66KA",
        "67KA",
        "91KA",
        "92KA"
      ],
      "planogramStore": [
        {
          "storeCode": "16KA",
          "store_code": "16KA",
          "quantity": 6
        },
        {
          "storeCode": "17KA",
          "store_code": "17KA",
          "quantity": 6
        },
        {
          "storeCode": "30KA",
          "store_code": "30KA",
          "quantity": 6
        },
        {
          "storeCode": "32KA",
          "store_code": "32KA",
          "quantity": 6
        },
        {
          "storeCode": "34KA",
          "store_code": "34KA",
          "quantity": 6
        }
      ],
      "line_number": 1
    }
  ],
  "MATERIALGROUPINGS": [
    {
      "line_number": 1,
      "running_number": "1",
      "grouping_name": "MM8D1C22C1",
      "description": "MM8D1C22C1",
      "category": "1 - Material No.",
      "component": "1000728050"
    }
  ],
  "STATUS": "PENDING_VERIFICATION"
}
```

### C JSON

```json
{
  "LAYOUT": "C",
  "HEADER": {
    "theme": "C011",
    "promotionName": "M Price 09/2026",
    "group": "MERCHANDISE SUPERMARKET(GROCERY FOOD)",
    "purchasingGroup": "C02",
    "contractType": "",
    "bonusBuyProfile": "D002",
    "rebateChargeback": "",
    "singleMultiple": "Multiple",
    "wbsNumber": "AP.26.8883.10.CP.01",
    "volume": "",
    "vendorCode": "",
    "vendorName": "",
    "periodFrom": "2026-09-19",
    "periodTo": "2026-09-29",
    "timeFrom": "",
    "timeTo": "",
    "days": [
      "All"
    ],
    "status": "draft",
    "createdBy": "pornpat.pp@gmail.com",
    "created_by": "pornpat.pp@gmail.com",
    "updatedBy": "pornpat.pp@gmail.com",
    "updated_by": "pornpat.pp@gmail.com"
  },
  "BONUSBUYS": [],
  "CONDITIONS": [
    {
      "line_number": 0,
      "contract_number": "1",
      "conditionHeader": {
        "reason": "X",
        "contract_type": "Z200",
        "vendor": "MSH02",
        "vendor_name": "บริษัท เอ็ม. เอส. ฮานาโซโน",
        "department_mer": "Mer C",
        "start": "19.09.2026",
        "end": "29.09.2026",
        "payment_method": "M",
        "sales_organization": "2009",
        "settlement_option": "1",
        "header_text": "M-Price",
        "additional_text": "M-Price",
        "department": "DEP033"
      },
      "businessVolumePurchase": [
        {
          "set_of_field_combination": "BVCU",
          "field_combination": "Z256",
          "text_field_combination": "Vendor, Material, Cond. Type, Sales Unit",
          "inclusive_exclusive": "Inclusive",
          "contract_number": "1"
        }
      ],
      "businessVolumeSales": [
        {
          "set_of_field_combination": "BVCU",
          "field_combination": "Z256",
          "text_field_combination": "Vendor, Material, Cond. Type, Sales Unit",
          "inclusive_exclusive": "Inclusive",
          "selection_group": "Z001",
          "vendor": "MSH02",
          "vendor_name": "บริษัท เอ็ม. เอส. ฮานาโซโน",
          "bonus_buy": "1",
          "contract_number": "1"
        }
      ],
      "conditionType": [
        {
          "condition_table": "V 163",
          "condition_type": "1",
          "brand_name": "บริษัท เอ็ม. เอส. ฮานาโซโน",
          "valid_from": "19.09.2026",
          "valid_to": "29.09.2026",
          "condition_rate": 1,
          "condition_unit": "1",
          "scale_unit": "EA",
          "contract_number": "1"
        }
      ],
      "settlementCalendar": [],
      "combineCheck": [],
      "allocation": []
    }
  ],
  "MATERIALS": [
    {
      "line_number": 0,
      "number_of_promotion": 1,
      "number_of_material_grouping": 1,
      "mch": "HBA",
      "purchasing_group": "C05",
      "material": "1000728050",
      "barcode": "4901008314037",
      "material_description_th": "แคนเมค มาร์ชเมลโลว์พาวเดอร์อะบลูม#01",
      "material_description_en": "CANMAKE MARSHMALLOW POWDER ABLOOM #01",
      "vendor": "MSH02",
      "vendor_description": "บริษัท เอ็ม. เอส. ฮานาโซโน",
      "flow_type": "DS",
      "pack_size": 1,
      "sales_unit": "EA",
      "sales_tax": "V",
      "cost_price_case_excl_vat_normal": 329.72,
      "cost_price_unit_inc_vat_normal": 352.8,
      "cost_price_unit_inc_vat_promo": "352.8",
      "sales_price_normal": 490,
      "sales_price_promotion": 500,
      "bonus_buy_profile": "P010",
      "mechanic": "Price Off",
      "discount_percent_deal": "2.04%",
      "gross_profit_normal": "28%",
      "gross_profit_promotion": "29.44%",
      "forecast_quantity": "-",
      "forecast_amount": "-",
      "sales_quantity_1_month_ago": 8,
      "sales_quantity_2_months_ago": 4,
      "sales_quantity_3_months_ago": 7,
      "stores": [
        "13KA",
        "14KA",
        "15KA",
        "16KA",
        "17KA",
        "30KA",
        "31KA",
        "32KA",
        "33KA",
        "34KA",
        "60KA",
        "61KA",
        "64KA",
        "65KA",
        "66KA",
        "67KA",
        "91KA",
        "92KA"
      ],
      "planogramStore": [
        {
          "storeCode": "16KA",
          "store_code": "16KA",
          "quantity": 6
        },
        {
          "storeCode": "17KA",
          "store_code": "17KA",
          "quantity": 6
        },
        {
          "storeCode": "30KA",
          "store_code": "30KA",
          "quantity": 6
        },
        {
          "storeCode": "32KA",
          "store_code": "32KA",
          "quantity": 6
        },
        {
          "storeCode": "34KA",
          "store_code": "34KA",
          "quantity": 6
        }
      ]
    }
  ],
  "MATERIALGROUPINGS": [],
  "STATUS": "PENDING_VERIFICATION"
}
```
