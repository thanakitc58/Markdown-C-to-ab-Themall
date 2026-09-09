# P010 / Price Off / Promo baseline

```text
AB JSON                                                                            | C JSON
-----------------------------------------------------------------------------------+-----------------------------------------
{                                                                                  | {
  "LAYOUT": "AB",                                                                  |   "LAYOUT": "C",
  "HEADER": {                                                                      |   "HEADER": {
    "group": "MERCHANDISE SUPERMARKET(GROCERY FOOD)",                              |     "theme": "C011",
    "promotionName": "M Price 09/2026",                                            |     "promotionName": "M Price 09/2026",
    "purchasingGroup": "C02",                                                      |     "group": "MERCHANDISE SUPERMARKET(GROCERY FOOD)",
    "theme": "C011",                                                               |     "purchasingGroup": "C02",
    "bonusBuyProfile": "D002",                                                     |     "contractType": "",
    "rebateChargeback": "",                                                        |     "bonusBuyProfile": "D002",
    "contractType": "",                                                            |     "rebateChargeback": "",
    "wbsNumber": "AP.26.8883.10.CP.01",                                            |     "singleMultiple": "Multiple",
    "vendorCode": "NOBP",                                                          |     "wbsNumber": "AP.26.8883.10.CP.01",
    "vendorName": "",                                                              |     "volume": "",
    "periodFrom": "2026-09-19",                                                    |     "vendorCode": "",
    "periodTo": "2026-09-29",                                                      |     "vendorName": "",
    "days": [                                                                      |     "periodFrom": "2026-09-19",
      "All"                                                                        |     "periodTo": "2026-09-29",
    ],                                                                             |     "timeFrom": "",
    "singleMultiple": "Multiple",                                                  |     "timeTo": "",
    "volume": "",                                                                  |     "days": [
    "timeFrom": "",                                                                |       "All"
    "timeTo": ""                                                                   |     ],
  },                                                                               |     "status": "draft",
  "BONUSBUYS": [                                                                   |     "createdBy": "pornpat.pp@gmail.com",
    {                                                                              |     "created_by": "pornpat.pp@gmail.com",
      "line_number": 1,                                                            |     "updatedBy": "pornpat.pp@gmail.com",
      "bonus_buy_number": "1",                                                     |     "updated_by": "pornpat.pp@gmail.com"
      "bonusBuyHeader": {                                                          |   },
        "bonus_buy_number": "1",                                                   |   "BONUSBUYS": [],
        "bonus_buy_profile": "P010",                                               |   "CONDITIONS": [
        "mechanic": "Price Off",                                                   |     {
        "wbs_number": "AP.26.8883.10.CP.01",                                       |       "line_number": 0,
        "reference_code": "",                                                      |       "contract_number": "1",
        "promotion_area": "P1"                                                     |       "conditionHeader": {
      },                                                                           |         "reason": "X",
      "buy": [],                                                                   |         "contract_type": "Z200",
      "get": [                                                                     |         "vendor": "MSH02",
        {                                                                          |         "vendor_name": "บริษัท เอ็ม. เอส. ฮานาโซโน",
          "bonus_buy_number": "1",                                                 |         "department_mer": "Mer C",
          "material_or_group_type": "Material Group",                              |         "start": "19.09.2026",
          "material_or_group_code": "MM8D1C22C1",                                  |         "end": "29.09.2026",
          "get_quantity": 1,                                                       |         "payment_method": "M",
          "material_or_group_description": "แคนเมค มาร์ชเมลโลว์พาวเดอร์อะบลูม#01", |         "sales_organization": "2009",
          "sap_master_description": "CANMAKE MARSHMALLOW POWDER ABLOOM #01",       |         "settlement_option": "1",
          "normal_cost_price_excluding_vat": "329.72",                             |         "header_text": "M-Price",
          "normal_sales_price": "490",                                             |         "additional_text": "M-Price",
          "promotion_cost_price_excluding_vat": "352.8",                           |         "department": "DEP033"
          "promotion_price_thb": "500",                                            |       },
          "ean": "4901008314037",                                                  |       "businessVolumePurchase": [
          "unit": "EA"                                                             |         {
        }                                                                          |           "set_of_field_combination": "BVCU",
      ],                                                                           |           "field_combination": "Z256",
      "stores": [                                                                  |           "text_field_combination": "Vendor, Material, Cond. Type, Sales Unit",
        "13KA",                                                                    |           "inclusive_exclusive": "Inclusive",
        "14KA",                                                                    |           "contract_number": "1"
        "15KA",                                                                    |         }
        "16KA",                                                                    |       ],
        "17KA",                                                                    |       "businessVolumeSales": [
        "30KA",                                                                    |         {
        "31KA",                                                                    |           "set_of_field_combination": "BVCU",
        "32KA",                                                                    |           "field_combination": "Z256",
        "33KA",                                                                    |           "text_field_combination": "Vendor, Material, Cond. Type, Sales Unit",
        "34KA",                                                                    |           "inclusive_exclusive": "Inclusive",
        "60KA",                                                                    |           "selection_group": "Z001",
        "61KA",                                                                    |           "vendor": "MSH02",
        "64KA",                                                                    |           "vendor_name": "บริษัท เอ็ม. เอส. ฮานาโซโน",
        "65KA",                                                                    |           "bonus_buy": "1",
        "66KA",                                                                    |           "contract_number": "1"
        "67KA",                                                                    |         }
        "91KA",                                                                    |       ],
        "92KA"                                                                     |       "conditionType": [
      ],                                                                           |         {
      "card": [],                                                                  |           "condition_table": "V 163",
      "tender": [],                                                                |           "condition_type": "1",
      "installment": [],                                                           |           "brand_name": "บริษัท เอ็ม. เอส. ฮานาโซโน",
      "posTerminal": [],                                                           |           "valid_from": "19.09.2026",
      "premium": [],                                                               |           "valid_to": "29.09.2026",
      "coupon": [],                                                                |           "condition_rate": 1,
      "limitControl": []                                                           |           "condition_unit": "1",
    }                                                                              |           "scale_unit": "EA",
  ],                                                                               |           "contract_number": "1"
  "CONDITIONS": [                                                                  |         }
    {                                                                              |       ],
      "line_number": 0,                                                            |       "settlementCalendar": [],
      "contract_number": "1",                                                      |       "combineCheck": [],
      "conditionHeader": {                                                         |       "allocation": []
        "reason": "X",                                                             |     }
        "contract_type": "Z200",                                                   |   ],
        "vendor": "MSH02",                                                         |   "MATERIALS": [
        "vendor_name": "บริษัท เอ็ม. เอส. ฮานาโซโน",                               |     {
        "department_mer": "Mer C",                                                 |       "line_number": 0,
        "start": "19.09.2026",                                                     |       "number_of_promotion": 1,
        "end": "29.09.2026",                                                       |       "number_of_material_grouping": 1,
        "payment_method": "M",                                                     |       "mch": "HBA",
        "sales_organization": "2009",                                              |       "purchasing_group": "C05",
        "settlement_option": "1",                                                  |       "material": "1000728050",
        "header_text": "M-Price",                                                  |       "barcode": "4901008314037",
        "additional_text": "M-Price",                                              |       "material_description_th": "แคนเมค มาร์ชเมลโลว์พาวเดอร์อะบลูม#01",
        "department": "DEP033"                                                     |       "material_description_en": "CANMAKE MARSHMALLOW POWDER ABLOOM #01",
      },                                                                           |       "vendor": "MSH02",
      "businessVolumePurchase": [                                                  |       "vendor_description": "บริษัท เอ็ม. เอส. ฮานาโซโน",
        {                                                                          |       "flow_type": "DS",
          "set_of_field_combination": "BVCU",                                      |       "pack_size": 1,
          "field_combination": "Z256",                                             |       "sales_unit": "EA",
          "text_field_combination": "Vendor, Material, Cond. Type, Sales Unit",    |       "sales_tax": "V",
          "inclusive_exclusive": "Inclusive",                                      |       "cost_price_case_excl_vat_normal": 329.72,
          "contract_number": "1"                                                   |       "cost_price_unit_inc_vat_normal": 352.8,
        }                                                                          |       "cost_price_unit_inc_vat_promo": "352.8",
      ],                                                                           |       "sales_price_normal": 490,
      "businessVolumeSales": [                                                     |       "sales_price_promotion": 500,
        {                                                                          |       "bonus_buy_profile": "P010",
          "set_of_field_combination": "BVCU",                                      |       "mechanic": "Price Off",
          "field_combination": "Z256",                                             |       "discount_percent_deal": "2.04%",
          "text_field_combination": "Vendor, Material, Cond. Type, Sales Unit",    |       "gross_profit_normal": "28%",
          "inclusive_exclusive": "Inclusive",                                      |       "gross_profit_promotion": "29.44%",
          "selection_group": "Z001",                                               |       "forecast_quantity": "-",
          "vendor": "MSH02",                                                       |       "forecast_amount": "-",
          "vendor_name": "บริษัท เอ็ม. เอส. ฮานาโซโน",                             |       "sales_quantity_1_month_ago": 8,
          "bonus_buy": "1",                                                        |       "sales_quantity_2_months_ago": 4,
          "contract_number": "1"                                                   |       "sales_quantity_3_months_ago": 7,
        }                                                                          |       "stores": [
      ],                                                                           |         "13KA",
      "conditionType": [                                                           |         "14KA",
        {                                                                          |         "15KA",
          "condition_table": "V 163",                                              |         "16KA",
          "condition_type": "1",                                                   |         "17KA",
          "brand_name": "บริษัท เอ็ม. เอส. ฮานาโซโน",                              |         "30KA",
          "valid_from": "19.09.2026",                                              |         "31KA",
          "valid_to": "29.09.2026",                                                |         "32KA",
          "condition_rate": 1,                                                     |         "33KA",
          "condition_unit": "1",                                                   |         "34KA",
          "scale_unit": "EA",                                                      |         "60KA",
          "contract_number": "1"                                                   |         "61KA",
        }                                                                          |         "64KA",
      ],                                                                           |         "65KA",
      "settlementCalendar": [],                                                    |         "66KA",
      "combineCheck": [],                                                          |         "67KA",
      "allocation": []                                                             |         "91KA",
    }                                                                              |         "92KA"
  ],                                                                               |       ],
  "MATERIALS": [                                                                   |       "planogramStore": [
    {                                                                              |         {
      "number_of_promotion": 1,                                                    |           "storeCode": "16KA",
      "number_of_material_grouping": 1,                                            |           "store_code": "16KA",
      "mch": "HBA",                                                                |           "quantity": 6
      "purchasing_group": "C05",                                                   |         },
      "material": "1000728050",                                                    |         {
      "barcode": "4901008314037",                                                  |           "storeCode": "17KA",
      "material_description_th": "แคนเมค มาร์ชเมลโลว์พาวเดอร์อะบลูม#01",           |           "store_code": "17KA",
      "material_description_en": "CANMAKE MARSHMALLOW POWDER ABLOOM #01",          |           "quantity": 6
      "vendor": "MSH02",                                                           |         },
      "vendor_description": "บริษัท เอ็ม. เอส. ฮานาโซโน",                          |         {
      "flow_type": "DS",                                                           |           "storeCode": "30KA",
      "pack_size": 1,                                                              |           "store_code": "30KA",
      "sales_unit": "EA",                                                          |           "quantity": 6
      "sales_tax": "V",                                                            |         },
      "cost_price_case_excl_vat_normal": 329.72,                                   |         {
      "cost_price_unit_inc_vat_normal": 352.8,                                     |           "storeCode": "32KA",
      "cost_price_unit_inc_vat_promo": "352.8",                                    |           "store_code": "32KA",
      "sales_price_normal": 490,                                                   |           "quantity": 6
      "sales_price_promotion": 500,                                                |         },
      "bonus_buy_profile": "P010",                                                 |         {
      "mechanic": "Price Off",                                                     |           "storeCode": "34KA",
      "discount_percent_deal": "2.04%",                                            |           "store_code": "34KA",
      "gross_profit_normal": "28%",                                                |           "quantity": 6
      "gross_profit_promotion": "29.44%",                                          |         }
      "forecast_quantity": "-",                                                    |       ]
      "forecast_amount": "-",                                                      |     }
      "sales_quantity_1_month_ago": 8,                                             |   ],
      "sales_quantity_2_months_ago": 4,                                            |   "MATERIALGROUPINGS": [],
      "sales_quantity_3_months_ago": 7,                                            |   "STATUS": "PENDING_VERIFICATION"
      "stores": [                                                                  | }
        "13KA",                                                                    | 
        "14KA",                                                                    | 
        "15KA",                                                                    | 
        "16KA",                                                                    | 
        "17KA",                                                                    | 
        "30KA",                                                                    | 
        "31KA",                                                                    | 
        "32KA",                                                                    | 
        "33KA",                                                                    | 
        "34KA",                                                                    | 
        "60KA",                                                                    | 
        "61KA",                                                                    | 
        "64KA",                                                                    | 
        "65KA",                                                                    | 
        "66KA",                                                                    | 
        "67KA",                                                                    | 
        "91KA",                                                                    | 
        "92KA"                                                                     | 
      ],                                                                           | 
      "planogramStore": [                                                          | 
        {                                                                          | 
          "storeCode": "16KA",                                                     | 
          "store_code": "16KA",                                                    | 
          "quantity": 6                                                            | 
        },                                                                         | 
        {                                                                          | 
          "storeCode": "17KA",                                                     | 
          "store_code": "17KA",                                                    | 
          "quantity": 6                                                            | 
        },                                                                         | 
        {                                                                          | 
          "storeCode": "30KA",                                                     | 
          "store_code": "30KA",                                                    | 
          "quantity": 6                                                            | 
        },                                                                         | 
        {                                                                          | 
          "storeCode": "32KA",                                                     | 
          "store_code": "32KA",                                                    | 
          "quantity": 6                                                            | 
        },                                                                         | 
        {                                                                          | 
          "storeCode": "34KA",                                                     | 
          "store_code": "34KA",                                                    | 
          "quantity": 6                                                            | 
        }                                                                          | 
      ],                                                                           | 
      "line_number": 1                                                             | 
    }                                                                              | 
  ],                                                                               | 
  "MATERIALGROUPINGS": [                                                           | 
    {                                                                              | 
      "line_number": 1,                                                            | 
      "running_number": "1",                                                       | 
      "grouping_name": "MM8D1C22C1",                                               | 
      "description": "MM8D1C22C1",                                                 | 
      "category": "1 - Material No.",                                              | 
      "component": "1000728050"                                                    | 
    }                                                                              | 
  ],                                                                               | 
  "STATUS": "PENDING_VERIFICATION"                                                 | 
}                                                                                  | 
```
