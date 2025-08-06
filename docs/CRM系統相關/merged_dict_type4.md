---
sidebar_position: 1
---

0、前言
------
:::tip
**寫此篇文章的動機**  
這個檔案會介紹，如何讓兩個不同工作表(月份)的 sheet 資料融合成一個
:::



1、不透過發票號碼，透過 unique key 合併
------

* 兩個月份的格式都一樣，不一樣的只有裡面的內容，11月份中有的資料，10月份可能只有幾個有

```py
def merged_two_month_op_sheet():
    # 先取得 11 月份，當月的所有數據
    NOV_month_OPSHEET_data = filter_current_google_sheet_data("TW-11月")
    # 先取得 10 月份，當月的所有數據
    OCT_month_OPSHEET_data = filter_other_google_sheet_data("TW-10月")

    # 要更新的欄位清單
    fields_to_update = [
        "總走期累積收費",
        "總走期累積成本",
        "OP表_累積毛利",
        "OP表_毛利率",
        "OP表_FB花費",
        "OP表_GADS花費",
        "OP表_TIKTOK花費",
        "預估當月毛利",
    ]    

    # 遍歷 11 月份的資料
    for invoice, op_sheet_details in NOV_month_OPSHEET_data.items():
        
        # 從 11 月份的資料取出 unique key
        unique_key = op_sheet_details.get("獨特KEY值")        
        
        # 如果 10 月有也跟 11 月同樣的 unique key，我們來把 10 月的資料更新進 11 月的 dict 中
        if unique_key in OCT_month_OPSHEET_data:
            # 先遍歷剛剛要更新的欄位
            for field in fields_to_update:
                # 確認和取得該筆資料在 10 月份的欄位和內容
                if field in OCT_month_OPSHEET_data[unique_key]:
                    # 用 10 月份的欄位，更新 11 月份的欄位
                    op_sheet_details[field] = OCT_month_OPSHEET_data[unique_key][field]    

    # 最後回傳 11 月份資料
    return NOV_month_OPSHEET_data
```