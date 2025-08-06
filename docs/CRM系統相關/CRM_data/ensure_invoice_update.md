---
sidebar_position: 1
---

0、前言
------
:::tip
**寫此篇文章的動機**  
這裡我想要確認發票更新是否成功，因此加上一些 log
:::


0、發票更新流程
------

* 傳進來的 data 是 11 月份的發票資料
* dashboard_and_op_sheet_data 是 11 月份的 dashboard 資料和指定的 op_sheet 資料合併
* 創建一個 flattened_dashboard_data dict
* 把 dashboard_and_op_sheet_data 的發票資料抓出來，組合新的 dict，資料格式為 `{"invoice": value}`

* 設定一個空的 set() -> matched_invoice_numbers，等等有更新成功的話，全部丟進來
* 遍歷 data(CRM) 資料，要抓出兩個資料，一個是 `發票資料`，一個是該筆的 `record_id`
* 在迴圈內，多加一個判斷式 -> `matching_record = flattened_dashboard_data.get(invoice_number)`，如果今天 matching_record 為 `true`，代表這筆發票可以更新
* 最後用 `所有的發票數量 - 更新過的發票 = 沒有被更新的發票 -> unmatched_invoice_numbers` 就拿到了哪些發票沒有被更新




1、function - bulk_update_invoice_data
------


```py
def bulk_update_invoice_data(data):
    access_token = get_access_token()
    
    start_date = "2024-11-01"
    end_date = "2024-11-30"
    # 拿到 11 月份的 dashboard 資料 -------------- step 2 ---------------
    dashboard_and_op_sheet_data = merged_op_and_dashboard_data(start_date, end_date)

    # 創建一個空的 {}，等等把資料塞進此 dict 中 -------------- step 3 ---------------
    flattened_dashboard_data = {}
    
    for key, value in dashboard_and_op_sheet_data.items():
        for date_range, details in value.items():
            invoice_number = details.get("發票號碼", "")
            if invoice_number:
                flattened_dashboard_data[invoice_number] = details
    
    # 創建一個空的 set()，如果等等有發票被更新，加進來這個 set 中 -------------- step 4 ---------------
    matched_invoice_numbers = set()
    
    # 遍歷 CRM 資料並檢查發票，這個 data 就是 CRM 的 11 月發票 -------------- step 5 ---------------
    for key, value in data.items():        
        print(f"Processing CRM Record ID: {key}")
        
        record_id = key
        module_name = "CustomModule22"
        url = f"https://www.zohoapis.com/crm/v2/{module_name}/{record_id}"
        
        invoice_number = value.get("invoice", {}).get("發票編號", "")
        print(f"Invoice Number: {invoice_number}")
        
        # 查找能否在 flattened_dashboard_data 中，找到剛剛遍歷的 CRM 發票 number -------------- step 6 ---------------
        matching_record = flattened_dashboard_data.get(invoice_number)
        
        # 如果有的話，就更新該筆發票 -------------- step 7 ---------------
        if matching_record:
            
            # 並且做剛剛說的，把有 match 的發票丟進 set 中  -------------- step 8 ---------------
            matched_invoice_numbers.add(invoice_number)            
            print(f"發票號碼: {invoice_number}")

            # 使用匹配的資料來更新 CRM -------------- step 9 ---------------
            updated_data = {
                "Meta": matching_record.get("Dashboard_FB花費", 0),
                "Google": matching_record.get("Dashboard_GADS花費", 0),
                "TikTok": matching_record.get("Dashboard_Tiktok花費", 0),
                "OP": matching_record.get("OP表_FB花費", 0),
                "Google_OP": matching_record.get("OP表_GADS花費", 0),
                "TikTok_OP": matching_record.get("OP表_TIKTOK花費", 0),
            }
            
            headers = {
                "Authorization": f"Zoho-oauthtoken {access_token}",
                "Content-Type": "application/json"
            }

            response = requests.patch(url, headers=headers, json={"data": [updated_data]})

            # 檢查回應
            if response.status_code == 200:
                print("資料更新成功！")
                print(response.json())
            else:
                print("資料更新失敗！")
                print(response.json())
        else:
            print(f"未找到匹配的發票號碼: {invoice_number}")
    
    # 用 flattened_dashboard_data 和 matched_invoice_numbers 得到哪些發票沒有被更新成功 -------------- step 10 ---------------
    unmatched_invoice_numbers = set(flattened_dashboard_data.keys()) - matched_invoice_numbers
    print("\n未被更新的發票清單:")

    # 把沒有更新成功的所有的發票內容印出來 -------------- step 11 ---------------
    for invoice_number in unmatched_invoice_numbers:
        print(f"未更新的發票號碼: {invoice_number}")
        print(f"對應資料: {flattened_dashboard_data[invoice_number]}")


if __name__ == "__main__":
    # 這邊可以拿到11月的所有發票 -------------- step 1 ---------------
    start_date = "2024-11-01"
    end_date = "2024-11-30"
    filtered_data = filtered_CRM_data_by_date(start_date, end_date)
    bulk_update_invoice_data(filtered_data)
```