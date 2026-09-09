# F001 / B5G1 / Promo baseline

<table>
<thead>
<tr>
<th>AB JSON</th>
<th>C JSON</th>
</tr>
</thead>
<tbody>
<tr>
<td valign="top">

```json
{
  "LAYOUT": "AB",
  "HEADER": {
    "group": "MERCHANDISE SUPERMARKET(GROCERY FOOD)",
    "promotionName": "M Price 09/2026",
    "purchasingGroup": "C02",
    "theme": "C011",
    "bonusBuyProfile": "",
    "rebateChargeback": "",
    "contractType": "",
    "wbsNumber": "AP.26.8883.10.CP.01",
    "vendorCode": "NOBP",
    "vendorName": "Vendor not found",
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
        "bonus_buy_profile": "F001",
        "mechanic": "B5G1",
        "valid_time_from": "00:00:00",
        "valid_time_to": "23:59:59",
        "wbs_number": "AP.26.8883.10.CP.01",
        "promotion_area": "P1"
      },
      "buy": [
        {
          "bonus_buy_number": "1",
          "material_or_group_type": "Material Group",
          "material_or_group_code": "MM8D203141",
          "minimum_quantity": 5,
          "description": "แคนเมค มาร์ชเมลโลว์พาวเดอร์อะบลูม#01",
          "sap_master_description": "CANMAKE MARSHMALLOW POWDER ABLOOM #01",
          "normal_cost_price_excluding_vat": "329.72",
          "normal_sales_price": "490",
          "ean": "4901008314037",
          "sales_unit": "EA"
        }
      ],
      "get": [
        {
          "bonus_buy_number": "1",
          "material_or_group_type": "Material Group",
          "material_or_group_code": "MM8D203141",
          "get_quantity": 1,
          "material_or_group_description": "แคนเมค มาร์ชเมลโลว์พาวเดอร์อะบลูม#01",
          "sap_master_description": "CANMAKE MARSHMALLOW POWDER ABLOOM #01",
          "normal_cost_price_excluding_vat": "329.72",
          "promotion_cost_price_excluding_vat": "352.8",
          "promotion_price_thb": "1000",
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
      "line_number": 1,
      "contract_number": "1",
      "conditionHeader": {
        "reason": "1",
        "department_mer": "Mer C",
        "start": "19.09.2026",
        "end": "29.09.2026",
        "department": "DEP031"
      },
      "businessVolumePurchase": [
        {
          "inclusive_exclusive": "Inclusive",
          "contract_number": "1"
        }
      ],
      "businessVolumeSales": [
        {
          "inclusive_exclusive": "Inclusive",
          "selection_group": "Z001",
          "contract_number": "1"
        }
      ],
      "conditionType": [
        {
          "brand_name": "บริษัท เอ็ม. เอส. ฮานาโซโน",
          "valid_from": "19.09.2026",
          "valid_to": "29.09.2026",
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
      "line_number": 1,
      "number_of_promotion": "1",
      "number_of_material_grouping": "1",
      "mch": "HBA",
      "purchasing_group": "C02",
      "material": "1000728050",
      "barcode": "4901008314037",
      "material_description_th": "แคนเมค มาร์ชเมลโลว์พาวเดอร์อะบลูม#01",
      "material_description_en": "CANMAKE MARSHMALLOW POWDER ABLOOM #01",
      "vendor": "OIS05",
      "vendor_description": "บริษัท โออิชิ ฟู้ด เซอร์วิส",
      "flow_type": "DS",
      "pack_size": "1",
      "sales_unit": "EA",
      "sales_tax": "V",
      "cost_price_case_excl_vat_normal": "329.72",
      "cost_price_unit_inc_vat_normal": "352.8",
      "cost_price_unit_inc_vat_promo": "352.8",
      "sales_price_normal": "490",
      "sales_price_promotion": "1000",
      "bonus_buy_profile": "F001",
      "mechanic": "B5G1",
      "discount_percent_deal": "65.99%",
      "gross_profit_normal": "28%",
      "gross_profit_promotion": "13.6%",
      "forecast_quantity": "-",
      "forecast_amount": "-",
      "sales_quantity_1_month_ago": "8",
      "sales_quantity_2_months_ago": "4",
      "sales_quantity_3_months_ago": "7",
      "display_promotion_m_price_category": "Skincare",
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
          "quantity": "6"
        },
        {
          "storeCode": "17KA",
          "store_code": "17KA",
          "quantity": "6"
        },
        {
          "storeCode": "30KA",
          "store_code": "30KA",
          "quantity": "6"
        },
        {
          "storeCode": "32KA",
          "store_code": "32KA",
          "quantity": "6"
        },
        {
          "storeCode": "34KA",
          "store_code": "34KA",
          "quantity": "6"
        }
      ]
    }
  ],
  "MATERIALGROUPINGS": [
    {
      "line_number": 1,
      "running_number": "1",
      "grouping_name": "MM8D203141",
      "description": "MM8D203141",
      "category": "1 - Material No.",
      "component": "1000728050"
    }
  ],
  "STATUS": "PENDING_VERIFICATION"
}
```

</td>
<td valign="top">

```json
{
  "LAYOUT": "C",
  "HEADER": {
    "theme": "C011",
    "promotionName": "M Price 09/2026",
    "group": "MERCHANDISE SUPERMARKET(GROCERY FOOD)",
    "purchasingGroup": "C02",
    "contractType": "",
    "bonusBuyProfile": "",
    "rebateChargeback": "",
    "singleMultiple": "Multiple",
    "wbsNumber": "AP.26.8883.10.CP.01",
    "volume": "",
    "vendorCode": "NOBP",
    "vendorName": "Vendor not found",
    "periodFrom": "2026-09-19",
    "periodTo": "2026-09-29",
    "timeFrom": "",
    "timeTo": "",
    "days": [
      "All"
    ],
    "status": "submitted",
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
        "reason": "1",
        "department_mer": "Mer C",
        "start": "19.09.2026",
        "end": "29.09.2026",
        "department": "DEP031"
      },
      "businessVolumePurchase": [
        {
          "inclusive_exclusive": "Inclusive",
          "contract_number": "1"
        }
      ],
      "businessVolumeSales": [
        {
          "inclusive_exclusive": "Inclusive",
          "selection_group": "Z001",
          "contract_number": "1"
        }
      ],
      "conditionType": [
        {
          "brand_name": "บริษัท เอ็ม. เอส. ฮานาโซโน",
          "valid_from": "19.09.2026",
          "valid_to": "29.09.2026",
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
      "number_of_promotion": "1",
      "number_of_material_grouping": "1",
      "mch": "HBA",
      "purchasing_group": "C02",
      "material": "1000728050",
      "barcode": "4901008314037",
      "material_description_th": "แคนเมค มาร์ชเมลโลว์พาวเดอร์อะบลูม#01",
      "material_description_en": "CANMAKE MARSHMALLOW POWDER ABLOOM #01",
      "vendor": "OIS05",
      "vendor_description": "บริษัท โออิชิ ฟู้ด เซอร์วิส",
      "flow_type": "DS",
      "pack_size": "1",
      "sales_unit": "EA",
      "sales_tax": "V",
      "cost_price_case_excl_vat_normal": "329.72",
      "cost_price_unit_inc_vat_normal": "352.8",
      "cost_price_unit_inc_vat_promo": "352.8",
      "sales_price_normal": "490",
      "sales_price_promotion": "1000",
      "bonus_buy_profile": "F001",
      "mechanic": "B5G1",
      "discount_percent_deal": "65.99%",
      "gross_profit_normal": "28%",
      "gross_profit_promotion": "13.6%",
      "forecast_quantity": "-",
      "forecast_amount": "-",
      "sales_quantity_1_month_ago": "8",
      "sales_quantity_2_months_ago": "4",
      "sales_quantity_3_months_ago": "7",
      "display_promotion_m_price_category": "Skincare",
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
          "quantity": "6"
        },
        {
          "storeCode": "17KA",
          "store_code": "17KA",
          "quantity": "6"
        },
        {
          "storeCode": "30KA",
          "store_code": "30KA",
          "quantity": "6"
        },
        {
          "storeCode": "32KA",
          "store_code": "32KA",
          "quantity": "6"
        },
        {
          "storeCode": "34KA",
          "store_code": "34KA",
          "quantity": "6"
        }
      ]
    }
  ],
  "MATERIALGROUPINGS": [],
  "STATUS": "PENDING_VERIFICATION"
}
```

</td>
</tr>
</tbody>
</table>
