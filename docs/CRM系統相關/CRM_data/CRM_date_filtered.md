---
sidebar_position: 1
---

0、前言
------
:::tip
**寫此篇文章的動機**  
加上時間參數，拿到特定區間的發票資料
:::


1、取得 invoice
------

* 假如果們現在已經可以得到一批這類格式的資料

```py
invoice_data = 
{'5244876000037057151': 
   {'OP表資料': {'Facebook': None,
                'Google': None,
                'TikTok': None},
    'id': '5244876000037057151',
    'invoice': {'公司名稱': 
                   {'id': '5244876000030449127',
                    'name': '中保防災科技股份有限公司'},
                '合約及委刊': 
                   {'id': '5244876000030604001',
                    'name': '202409041190'},
                '廣告實際走期': '2024/12/1-2024/12/31',
                '廣告費用含服(未稅)': 71429,
                '最終發票金額(未稅)': 71429,
                '發票品名': ['廣告服務'],
                '發票編號': 'HF43182017',
                '發票開立日期': '2024-12-31',
                '貨幣': 'NT$',
                '開發票的人': 
                   {'email': 'marty@tagtoo.com',
                    'id': '5244876000001562019',
                    'name': 'Marty Chuang'},
                '預期收款日期': '2025-01-31'},
   '後台資料': {'Facebook': None,
               'Google': None,
               'TikTok': None},
   '毛利': {'OP 表_毛利': None,
           'OP 表_毛利率': None,
           '毛利': None,
           '毛利率': None}}
}
```

* 我們可以利用 `發票開立日期` 這一個值，來篩選特定區間的發票

```py

def filtered_CRM_data_by_date(start_date, end_date):
    start_date = datetime.strptime(start_date, "%Y-%m-%d")
    end_date = datetime.strptime(end_date, "%Y-%m-%d")

    # 這個代表所有的發票資料
    all_invoice_data = get_all_invoice_data()

    filtered_data_by_date = {}

    for record_id, invoice_details in all_invoice_data.items():
        # 取得 '發票開立日期'
        invoice_create_date = invoice_details.get("invoice", "").get("發票開立日期", "")

        if invoice_create_date != None:
            try:
                # 把抓到的日期格式轉成指定格式
                formatted_invoice_create_date = datetime.strptime(invoice_create_date, "%Y-%m-%d")

                # 如果時間有符合在 start_date 和 end_date 之間的話
                if start_date <= formatted_invoice_create_date <= end_date:
                    
                    # 就重新把這區段的發票，丟進剛剛設定好的空 dict 中
                    filtered_data_by_date[record_id] = invoice_details
            
            except ValueError:
                print(f"Invalid date format for record {record_id}: {invoice_create_date}")
                continue
    
    return filtered_data_by_date
```

