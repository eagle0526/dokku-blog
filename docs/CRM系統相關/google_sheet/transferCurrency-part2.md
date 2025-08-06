---
sidebar_position: 2
---

0、前言
------
:::tip
**寫此篇文章的動機**  
這邊會整理所有 google sheet 讀取和整理成 dict 格式的相關 function
:::


1、檢視剛剛的 dict
------

* 剛剛在 sheetToDict 最後的結構資料會像這樣，但是這有個問題，就是金錢的格式要改成純數字

```py
result_op_data = {
    '11月待開': {'ec_id': '225',
               '客戶名稱': '寰宇迪士尼_數網業外',
               '實際廣告花費走期': '10/1~10/31',
               '實際開發票金額(未稅)': "11月待開",               
               '發票號碼': '11月待開',               
               '預估當月毛利': NT$7,844},
    'FG41999157': {'ec_id': '946',
               '客戶名稱': '上通電子',
               '實際廣告花費走期': '9/18~10/7',
               '實際開發票金額(未稅)': NT$14,281,               
               '發票號碼': 'FG41999157',               
               '預估當月毛利': NT$25,433},
               }
```

2、修改剛剛的function
------

```py
def transfer_sheet_to_dict():    
    op_sheet_data = get_google_sheet_data()    
    max_columns = max(len(row) for row in op_sheet_data)      
    columns = [[] for _ in range(max_columns)]

    for row in op_sheet_data:
        for index in range(max_columns):
            try:
                columns[index].append(row[index])
            except:
                columns[index].append("")
    
    headers = ["客戶名稱", "EC_ID", "發票號碼", "實際開發票金額(未稅)", "實際廣告花費走期"]
    indices = [0, 1, 3, 4, 5]

    result_op_data = {}
    
    # 這邊新增指定要轉換格式的欄位
    numeric_fields = {"實際開發票金額(未稅)"}


    for data in columns:
        try:
            row_data = { headers[i]: data[indices[i]] for i in range(len(headers))}            
            invoice = row_data.get('invoice', "").strip()

            if invoice:
                for key in numeric_fields:
                    # 判斷 row_data 中有沒有 numeric_fields 這些值，有的話就把這些值拿出來，並且把它轉成純數字
                    if key in row_data:                        
                        value = row_data[key]                        
                        # 正規表達式只拿數字
                        numeric_value = re.sub(r"[^\d]", "", value)
                        # 如果數字有效，轉成整數
                        row_data[key] = int(numeric_value) if numeric_value else 0

                result_op_data[invoice] = row_data
        except IndexError:
            print(f"資料不完整，跳過: {row}")
    
    return result_op_data
```


