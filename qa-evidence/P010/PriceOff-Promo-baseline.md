# P010 / Price Off / Promo baseline

> Tip: เปิดแบบ Wide ได้ที่ปุ่ม `Blame` ข้างๆ หรือซูมหน้าจอ — ตารางด้านล่างจัดชิดบนซ้าย-ขวาแล้ว

<table>
  <thead>
    <tr>
      <th width="50%">AB JSON</th>
      <th width="50%">C JSON</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td valign="top">
<pre><code>{
  &quot;LAYOUT&quot;: &quot;AB&quot;,
  &quot;HEADER&quot;: {
    &quot;group&quot;: &quot;MERCHANDISE SUPERMARKET(GROCERY FOOD)&quot;,
    &quot;promotionName&quot;: &quot;M Price 09/2026&quot;,
    &quot;purchasingGroup&quot;: &quot;C02&quot;,
    &quot;theme&quot;: &quot;C011&quot;,
    &quot;bonusBuyProfile&quot;: &quot;D002&quot;,
    &quot;rebateChargeback&quot;: &quot;&quot;,
    &quot;contractType&quot;: &quot;&quot;,
    &quot;wbsNumber&quot;: &quot;AP.26.8883.10.CP.01&quot;,
    &quot;vendorCode&quot;: &quot;NOBP&quot;,
    &quot;vendorName&quot;: &quot;&quot;,
    &quot;periodFrom&quot;: &quot;2026-09-19&quot;,
    &quot;periodTo&quot;: &quot;2026-09-29&quot;,
    &quot;days&quot;: [
      &quot;All&quot;
    ],
    &quot;singleMultiple&quot;: &quot;Multiple&quot;,
    &quot;volume&quot;: &quot;&quot;,
    &quot;timeFrom&quot;: &quot;&quot;,
    &quot;timeTo&quot;: &quot;&quot;
  },
  &quot;BONUSBUYS&quot;: [
    {
      &quot;line_number&quot;: 1,
      &quot;bonus_buy_number&quot;: &quot;1&quot;,
      &quot;bonusBuyHeader&quot;: {
        &quot;bonus_buy_number&quot;: &quot;1&quot;,
        &quot;bonus_buy_profile&quot;: &quot;P010&quot;,
        &quot;mechanic&quot;: &quot;Price Off&quot;,
        &quot;wbs_number&quot;: &quot;AP.26.8883.10.CP.01&quot;,
        &quot;reference_code&quot;: &quot;&quot;,
        &quot;promotion_area&quot;: &quot;P1&quot;
      },
      &quot;buy&quot;: [],
      &quot;get&quot;: [
        {
          &quot;bonus_buy_number&quot;: &quot;1&quot;,
          &quot;material_or_group_type&quot;: &quot;Material Group&quot;,
          &quot;material_or_group_code&quot;: &quot;MM8D1C22C1&quot;,
          &quot;get_quantity&quot;: 1,
          &quot;material_or_group_description&quot;: &quot;แคนเมค มาร์ชเมลโลว์พาวเดอร์อะบลูม#01&quot;,
          &quot;sap_master_description&quot;: &quot;CANMAKE MARSHMALLOW POWDER ABLOOM #01&quot;,
          &quot;normal_cost_price_excluding_vat&quot;: &quot;329.72&quot;,
          &quot;normal_sales_price&quot;: &quot;490&quot;,
          &quot;promotion_cost_price_excluding_vat&quot;: &quot;352.8&quot;,
          &quot;promotion_price_thb&quot;: &quot;500&quot;,
          &quot;ean&quot;: &quot;4901008314037&quot;,
          &quot;unit&quot;: &quot;EA&quot;
        }
      ],
      &quot;stores&quot;: [
        &quot;13KA&quot;,
        &quot;14KA&quot;,
        &quot;15KA&quot;,
        &quot;16KA&quot;,
        &quot;17KA&quot;,
        &quot;30KA&quot;,
        &quot;31KA&quot;,
        &quot;32KA&quot;,
        &quot;33KA&quot;,
        &quot;34KA&quot;,
        &quot;60KA&quot;,
        &quot;61KA&quot;,
        &quot;64KA&quot;,
        &quot;65KA&quot;,
        &quot;66KA&quot;,
        &quot;67KA&quot;,
        &quot;91KA&quot;,
        &quot;92KA&quot;
      ],
      &quot;card&quot;: [],
      &quot;tender&quot;: [],
      &quot;installment&quot;: [],
      &quot;posTerminal&quot;: [],
      &quot;premium&quot;: [],
      &quot;coupon&quot;: [],
      &quot;limitControl&quot;: []
    }
  ],
  &quot;CONDITIONS&quot;: [
    {
      &quot;line_number&quot;: 0,
      &quot;contract_number&quot;: &quot;1&quot;,
      &quot;conditionHeader&quot;: {
        &quot;reason&quot;: &quot;X&quot;,
        &quot;contract_type&quot;: &quot;Z200&quot;,
        &quot;vendor&quot;: &quot;MSH02&quot;,
        &quot;vendor_name&quot;: &quot;บริษัท เอ็ม. เอส. ฮานาโซโน&quot;,
        &quot;department_mer&quot;: &quot;Mer C&quot;,
        &quot;start&quot;: &quot;19.09.2026&quot;,
        &quot;end&quot;: &quot;29.09.2026&quot;,
        &quot;payment_method&quot;: &quot;M&quot;,
        &quot;sales_organization&quot;: &quot;2009&quot;,
        &quot;settlement_option&quot;: &quot;1&quot;,
        &quot;header_text&quot;: &quot;M-Price&quot;,
        &quot;additional_text&quot;: &quot;M-Price&quot;,
        &quot;department&quot;: &quot;DEP033&quot;
      },
      &quot;businessVolumePurchase&quot;: [
        {
          &quot;set_of_field_combination&quot;: &quot;BVCU&quot;,
          &quot;field_combination&quot;: &quot;Z256&quot;,
          &quot;text_field_combination&quot;: &quot;Vendor, Material, Cond. Type, Sales Unit&quot;,
          &quot;inclusive_exclusive&quot;: &quot;Inclusive&quot;,
          &quot;contract_number&quot;: &quot;1&quot;
        }
      ],
      &quot;businessVolumeSales&quot;: [
        {
          &quot;set_of_field_combination&quot;: &quot;BVCU&quot;,
          &quot;field_combination&quot;: &quot;Z256&quot;,
          &quot;text_field_combination&quot;: &quot;Vendor, Material, Cond. Type, Sales Unit&quot;,
          &quot;inclusive_exclusive&quot;: &quot;Inclusive&quot;,
          &quot;selection_group&quot;: &quot;Z001&quot;,
          &quot;vendor&quot;: &quot;MSH02&quot;,
          &quot;vendor_name&quot;: &quot;บริษัท เอ็ม. เอส. ฮานาโซโน&quot;,
          &quot;bonus_buy&quot;: &quot;1&quot;,
          &quot;contract_number&quot;: &quot;1&quot;
        }
      ],
      &quot;conditionType&quot;: [
        {
          &quot;condition_table&quot;: &quot;V 163&quot;,
          &quot;condition_type&quot;: &quot;1&quot;,
          &quot;brand_name&quot;: &quot;บริษัท เอ็ม. เอส. ฮานาโซโน&quot;,
          &quot;valid_from&quot;: &quot;19.09.2026&quot;,
          &quot;valid_to&quot;: &quot;29.09.2026&quot;,
          &quot;condition_rate&quot;: 1,
          &quot;condition_unit&quot;: &quot;1&quot;,
          &quot;scale_unit&quot;: &quot;EA&quot;,
          &quot;contract_number&quot;: &quot;1&quot;
        }
      ],
      &quot;settlementCalendar&quot;: [],
      &quot;combineCheck&quot;: [],
      &quot;allocation&quot;: []
    }
  ],
  &quot;MATERIALS&quot;: [
    {
      &quot;number_of_promotion&quot;: 1,
      &quot;number_of_material_grouping&quot;: 1,
      &quot;mch&quot;: &quot;HBA&quot;,
      &quot;purchasing_group&quot;: &quot;C05&quot;,
      &quot;material&quot;: &quot;1000728050&quot;,
      &quot;barcode&quot;: &quot;4901008314037&quot;,
      &quot;material_description_th&quot;: &quot;แคนเมค มาร์ชเมลโลว์พาวเดอร์อะบลูม#01&quot;,
      &quot;material_description_en&quot;: &quot;CANMAKE MARSHMALLOW POWDER ABLOOM #01&quot;,
      &quot;vendor&quot;: &quot;MSH02&quot;,
      &quot;vendor_description&quot;: &quot;บริษัท เอ็ม. เอส. ฮานาโซโน&quot;,
      &quot;flow_type&quot;: &quot;DS&quot;,
      &quot;pack_size&quot;: 1,
      &quot;sales_unit&quot;: &quot;EA&quot;,
      &quot;sales_tax&quot;: &quot;V&quot;,
      &quot;cost_price_case_excl_vat_normal&quot;: 329.72,
      &quot;cost_price_unit_inc_vat_normal&quot;: 352.8,
      &quot;cost_price_unit_inc_vat_promo&quot;: &quot;352.8&quot;,
      &quot;sales_price_normal&quot;: 490,
      &quot;sales_price_promotion&quot;: 500,
      &quot;bonus_buy_profile&quot;: &quot;P010&quot;,
      &quot;mechanic&quot;: &quot;Price Off&quot;,
      &quot;discount_percent_deal&quot;: &quot;2.04%&quot;,
      &quot;gross_profit_normal&quot;: &quot;28%&quot;,
      &quot;gross_profit_promotion&quot;: &quot;29.44%&quot;,
      &quot;forecast_quantity&quot;: &quot;-&quot;,
      &quot;forecast_amount&quot;: &quot;-&quot;,
      &quot;sales_quantity_1_month_ago&quot;: 8,
      &quot;sales_quantity_2_months_ago&quot;: 4,
      &quot;sales_quantity_3_months_ago&quot;: 7,
      &quot;stores&quot;: [
        &quot;13KA&quot;,
        &quot;14KA&quot;,
        &quot;15KA&quot;,
        &quot;16KA&quot;,
        &quot;17KA&quot;,
        &quot;30KA&quot;,
        &quot;31KA&quot;,
        &quot;32KA&quot;,
        &quot;33KA&quot;,
        &quot;34KA&quot;,
        &quot;60KA&quot;,
        &quot;61KA&quot;,
        &quot;64KA&quot;,
        &quot;65KA&quot;,
        &quot;66KA&quot;,
        &quot;67KA&quot;,
        &quot;91KA&quot;,
        &quot;92KA&quot;
      ],
      &quot;planogramStore&quot;: [
        {
          &quot;storeCode&quot;: &quot;16KA&quot;,
          &quot;store_code&quot;: &quot;16KA&quot;,
          &quot;quantity&quot;: 6
        },
        {
          &quot;storeCode&quot;: &quot;17KA&quot;,
          &quot;store_code&quot;: &quot;17KA&quot;,
          &quot;quantity&quot;: 6
        },
        {
          &quot;storeCode&quot;: &quot;30KA&quot;,
          &quot;store_code&quot;: &quot;30KA&quot;,
          &quot;quantity&quot;: 6
        },
        {
          &quot;storeCode&quot;: &quot;32KA&quot;,
          &quot;store_code&quot;: &quot;32KA&quot;,
          &quot;quantity&quot;: 6
        },
        {
          &quot;storeCode&quot;: &quot;34KA&quot;,
          &quot;store_code&quot;: &quot;34KA&quot;,
          &quot;quantity&quot;: 6
        }
      ],
      &quot;line_number&quot;: 1
    }
  ],
  &quot;MATERIALGROUPINGS&quot;: [
    {
      &quot;line_number&quot;: 1,
      &quot;running_number&quot;: &quot;1&quot;,
      &quot;grouping_name&quot;: &quot;MM8D1C22C1&quot;,
      &quot;description&quot;: &quot;MM8D1C22C1&quot;,
      &quot;category&quot;: &quot;1 - Material No.&quot;,
      &quot;component&quot;: &quot;1000728050&quot;
    }
  ],
  &quot;STATUS&quot;: &quot;PENDING_VERIFICATION&quot;
}</code></pre>
      </td>
      <td valign="top">
<pre><code>{
  &quot;LAYOUT&quot;: &quot;C&quot;,
  &quot;HEADER&quot;: {
    &quot;theme&quot;: &quot;C011&quot;,
    &quot;promotionName&quot;: &quot;M Price 09/2026&quot;,
    &quot;group&quot;: &quot;MERCHANDISE SUPERMARKET(GROCERY FOOD)&quot;,
    &quot;purchasingGroup&quot;: &quot;C02&quot;,
    &quot;contractType&quot;: &quot;&quot;,
    &quot;bonusBuyProfile&quot;: &quot;D002&quot;,
    &quot;rebateChargeback&quot;: &quot;&quot;,
    &quot;singleMultiple&quot;: &quot;Multiple&quot;,
    &quot;wbsNumber&quot;: &quot;AP.26.8883.10.CP.01&quot;,
    &quot;volume&quot;: &quot;&quot;,
    &quot;vendorCode&quot;: &quot;&quot;,
    &quot;vendorName&quot;: &quot;&quot;,
    &quot;periodFrom&quot;: &quot;2026-09-19&quot;,
    &quot;periodTo&quot;: &quot;2026-09-29&quot;,
    &quot;timeFrom&quot;: &quot;&quot;,
    &quot;timeTo&quot;: &quot;&quot;,
    &quot;days&quot;: [
      &quot;All&quot;
    ],
    &quot;status&quot;: &quot;draft&quot;,
    &quot;createdBy&quot;: &quot;pornpat.pp@gmail.com&quot;,
    &quot;created_by&quot;: &quot;pornpat.pp@gmail.com&quot;,
    &quot;updatedBy&quot;: &quot;pornpat.pp@gmail.com&quot;,
    &quot;updated_by&quot;: &quot;pornpat.pp@gmail.com&quot;
  },
  &quot;BONUSBUYS&quot;: [],
  &quot;CONDITIONS&quot;: [
    {
      &quot;line_number&quot;: 0,
      &quot;contract_number&quot;: &quot;1&quot;,
      &quot;conditionHeader&quot;: {
        &quot;reason&quot;: &quot;X&quot;,
        &quot;contract_type&quot;: &quot;Z200&quot;,
        &quot;vendor&quot;: &quot;MSH02&quot;,
        &quot;vendor_name&quot;: &quot;บริษัท เอ็ม. เอส. ฮานาโซโน&quot;,
        &quot;department_mer&quot;: &quot;Mer C&quot;,
        &quot;start&quot;: &quot;19.09.2026&quot;,
        &quot;end&quot;: &quot;29.09.2026&quot;,
        &quot;payment_method&quot;: &quot;M&quot;,
        &quot;sales_organization&quot;: &quot;2009&quot;,
        &quot;settlement_option&quot;: &quot;1&quot;,
        &quot;header_text&quot;: &quot;M-Price&quot;,
        &quot;additional_text&quot;: &quot;M-Price&quot;,
        &quot;department&quot;: &quot;DEP033&quot;
      },
      &quot;businessVolumePurchase&quot;: [
        {
          &quot;set_of_field_combination&quot;: &quot;BVCU&quot;,
          &quot;field_combination&quot;: &quot;Z256&quot;,
          &quot;text_field_combination&quot;: &quot;Vendor, Material, Cond. Type, Sales Unit&quot;,
          &quot;inclusive_exclusive&quot;: &quot;Inclusive&quot;,
          &quot;contract_number&quot;: &quot;1&quot;
        }
      ],
      &quot;businessVolumeSales&quot;: [
        {
          &quot;set_of_field_combination&quot;: &quot;BVCU&quot;,
          &quot;field_combination&quot;: &quot;Z256&quot;,
          &quot;text_field_combination&quot;: &quot;Vendor, Material, Cond. Type, Sales Unit&quot;,
          &quot;inclusive_exclusive&quot;: &quot;Inclusive&quot;,
          &quot;selection_group&quot;: &quot;Z001&quot;,
          &quot;vendor&quot;: &quot;MSH02&quot;,
          &quot;vendor_name&quot;: &quot;บริษัท เอ็ม. เอส. ฮานาโซโน&quot;,
          &quot;bonus_buy&quot;: &quot;1&quot;,
          &quot;contract_number&quot;: &quot;1&quot;
        }
      ],
      &quot;conditionType&quot;: [
        {
          &quot;condition_table&quot;: &quot;V 163&quot;,
          &quot;condition_type&quot;: &quot;1&quot;,
          &quot;brand_name&quot;: &quot;บริษัท เอ็ม. เอส. ฮานาโซโน&quot;,
          &quot;valid_from&quot;: &quot;19.09.2026&quot;,
          &quot;valid_to&quot;: &quot;29.09.2026&quot;,
          &quot;condition_rate&quot;: 1,
          &quot;condition_unit&quot;: &quot;1&quot;,
          &quot;scale_unit&quot;: &quot;EA&quot;,
          &quot;contract_number&quot;: &quot;1&quot;
        }
      ],
      &quot;settlementCalendar&quot;: [],
      &quot;combineCheck&quot;: [],
      &quot;allocation&quot;: []
    }
  ],
  &quot;MATERIALS&quot;: [
    {
      &quot;line_number&quot;: 0,
      &quot;number_of_promotion&quot;: 1,
      &quot;number_of_material_grouping&quot;: 1,
      &quot;mch&quot;: &quot;HBA&quot;,
      &quot;purchasing_group&quot;: &quot;C05&quot;,
      &quot;material&quot;: &quot;1000728050&quot;,
      &quot;barcode&quot;: &quot;4901008314037&quot;,
      &quot;material_description_th&quot;: &quot;แคนเมค มาร์ชเมลโลว์พาวเดอร์อะบลูม#01&quot;,
      &quot;material_description_en&quot;: &quot;CANMAKE MARSHMALLOW POWDER ABLOOM #01&quot;,
      &quot;vendor&quot;: &quot;MSH02&quot;,
      &quot;vendor_description&quot;: &quot;บริษัท เอ็ม. เอส. ฮานาโซโน&quot;,
      &quot;flow_type&quot;: &quot;DS&quot;,
      &quot;pack_size&quot;: 1,
      &quot;sales_unit&quot;: &quot;EA&quot;,
      &quot;sales_tax&quot;: &quot;V&quot;,
      &quot;cost_price_case_excl_vat_normal&quot;: 329.72,
      &quot;cost_price_unit_inc_vat_normal&quot;: 352.8,
      &quot;cost_price_unit_inc_vat_promo&quot;: &quot;352.8&quot;,
      &quot;sales_price_normal&quot;: 490,
      &quot;sales_price_promotion&quot;: 500,
      &quot;bonus_buy_profile&quot;: &quot;P010&quot;,
      &quot;mechanic&quot;: &quot;Price Off&quot;,
      &quot;discount_percent_deal&quot;: &quot;2.04%&quot;,
      &quot;gross_profit_normal&quot;: &quot;28%&quot;,
      &quot;gross_profit_promotion&quot;: &quot;29.44%&quot;,
      &quot;forecast_quantity&quot;: &quot;-&quot;,
      &quot;forecast_amount&quot;: &quot;-&quot;,
      &quot;sales_quantity_1_month_ago&quot;: 8,
      &quot;sales_quantity_2_months_ago&quot;: 4,
      &quot;sales_quantity_3_months_ago&quot;: 7,
      &quot;stores&quot;: [
        &quot;13KA&quot;,
        &quot;14KA&quot;,
        &quot;15KA&quot;,
        &quot;16KA&quot;,
        &quot;17KA&quot;,
        &quot;30KA&quot;,
        &quot;31KA&quot;,
        &quot;32KA&quot;,
        &quot;33KA&quot;,
        &quot;34KA&quot;,
        &quot;60KA&quot;,
        &quot;61KA&quot;,
        &quot;64KA&quot;,
        &quot;65KA&quot;,
        &quot;66KA&quot;,
        &quot;67KA&quot;,
        &quot;91KA&quot;,
        &quot;92KA&quot;
      ],
      &quot;planogramStore&quot;: [
        {
          &quot;storeCode&quot;: &quot;16KA&quot;,
          &quot;store_code&quot;: &quot;16KA&quot;,
          &quot;quantity&quot;: 6
        },
        {
          &quot;storeCode&quot;: &quot;17KA&quot;,
          &quot;store_code&quot;: &quot;17KA&quot;,
          &quot;quantity&quot;: 6
        },
        {
          &quot;storeCode&quot;: &quot;30KA&quot;,
          &quot;store_code&quot;: &quot;30KA&quot;,
          &quot;quantity&quot;: 6
        },
        {
          &quot;storeCode&quot;: &quot;32KA&quot;,
          &quot;store_code&quot;: &quot;32KA&quot;,
          &quot;quantity&quot;: 6
        },
        {
          &quot;storeCode&quot;: &quot;34KA&quot;,
          &quot;store_code&quot;: &quot;34KA&quot;,
          &quot;quantity&quot;: 6
        }
      ]
    }
  ],
  &quot;MATERIALGROUPINGS&quot;: [],
  &quot;STATUS&quot;: &quot;PENDING_VERIFICATION&quot;
}</code></pre>
      </td>
    </tr>
  </tbody>
</table>
