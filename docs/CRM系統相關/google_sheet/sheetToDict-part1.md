---
sidebar_position: 1
---

0、前言
------
:::tip
**寫此篇文章的動機**  
這邊會整理所有 google sheet 讀取和整理成 dict 格式的相關 function
:::


1、GoogleSheet 格式
------

* 以下是在 google sheet 呈現的形式，我們要來把他們轉成 dict 的格式，並且只取我們要的幾個欄位

|  | 寰宇迪士尼_數網業外 | 上通電子 |  上通電子 |  晶澈 | 
|----------|----------|----------|----------|----------|
| ECID   | 225   | 946   | 946   | 2857   |
| 窗口   | Jasper   | Nelson   | Nelson   | Nelson   | 
| 發票號碼   | 11月待開   | FG41999157   | FG41999487   | FG41999491   |
| 實際開發票金額(未稅)   | 11月待開   | NT$14,281   | NT$19,390  | NT$7,697   |
| 實際廣告花費走期   | 10/1~10/31   | 9/18~10/7   | 10/11-10/30   | 9/1-9/30   |


2、讀取 sheet 
------

```py
def get_google_sheet_data():
    # 指定您的服務帳戶金鑰 JSON 文件路徑，這個檔案內容，要去 GCloud 產生
    SERVICE_ACCOUNT_FILE = '/Users/yee0526/Desktop/TAGTOO/CRM-RD-Account/credentials.json'
    SCOPES = ['https://www.googleapis.com/auth/spreadsheets']
    creds = Credentials.from_service_account_file(
            SERVICE_ACCOUNT_FILE, scopes=SCOPES)    

    service = build('sheets', 'v4', credentials=creds)

    # 這一段 ID 是要放 sheet 連結中， /spreadsheets/d/'這一段'
    SPREADSHEET_ID = '1dvRIDOMk_1UC0imrtJeghjox2y2bKHeKEMT7fOpKBEU'    
    # 這個是 sheet 分頁的名稱
    RANGE_NAME = 'TW-10月'

    sheet = service.spreadsheets()    
    result = sheet.values().get(spreadsheetId=SPREADSHEET_ID,
                                range=RANGE_NAME).execute()
    values = result.get('values', [])    

    if not values:
        print('No data found.')

    return values
```


3、轉換成 dict
------

這邊要先提到一件事情：

* sheet 讀到的格式會像這樣，陣列會是橫向新增，不過目前我們單筆資料是直的，因此我們要改一下陣列的形式
```md
data = [[A1, B1, C1, D1......], [A2, B2, C2, D2......], [A3, B3, C3, D4.....]]
```
* 要改成這樣，以一個 column 為一個 array

```md
data = [[A1, A2, A3, A4......], [B1, B2, B3, B4......], [C1, C2, C3, C4.....]]
```

* 開始轉換
```py
def transfer_sheet_to_dict():
    # 1. 先拿到 sheet 資料
    op_sheet_data = get_google_sheet_data()

    # 2. 判斷最多有幾筆資料 -> [A1, B1 ...] -> 找出橫向最大值是多少
    max_columns = max(len(row) for row in op_sheet_data)  

    # 3. 用剛剛最大值的數量，建置該數量的 []，等等要把[A1, B1 ...] 直向資料塞進 [] 中
    columns = [[] for _ in range(max_columns)]

    # 4. 跑雙迴圈，把 A1, A2, A3 .... 這種形式，裝到 columns 這個陣列中
    for row in op_sheet_data:
        for index in range(max_columns):
            try:
                columns[index].append(row[index])
            except:
                columns[index].append("")
    
    # 設定標頭和這些資料在 columns 中的 index
    headers = ["客戶名稱", "EC_ID", "發票號碼", "實際開發票金額(未稅)", "實際廣告花費走期"]
    indices = [0, 1, 3, 4, 5]

    result_op_data = {}

    for data in columns:
        try:
            row_data = { headers[i]: data[indices[i]] for i in range(len(headers))}

            # 這個 row data 的資料會是這樣
            # {'客戶名稱': '雅丰診所_時尚', 'ec_id': '3076', '發票號碼': 'FG41999106', '實際開發票金額(未稅)': 'NT$34,203', '實際廣告花費走期': '9/1~9/30'} ****

            # 接下來我想要用 invoice 來當作我的 key 值，因此先取得 invoice
            invoice = row_data.get('invoice', "").strip()
            # 最後把 invoice 和 row data 組合起來
            result_op_data[invoice] = row_data

        except IndexError:
            print(f"資料不完整，跳過: {row}")
    
    return result_op_data
```


