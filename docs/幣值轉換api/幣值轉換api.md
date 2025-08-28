---
sidebar_position: 1
---

0、前言
------
:::tip
**寫此篇文章的動機**  
從政府給的資料，建立一個幣值轉換的api，來幫助產品轉換幣值的時候，可以透過該api轉變
:::

1、實作流程大綱
------

1. 先讀取政府提供的幣值轉換 api - https://data.gov.tw/dataset/6708
2. 利用 app script，把政府資料存進 google sheet 中 (google sheet 當作資料庫)
3. 讀取 google sheet 的資料，寫一隻 cloud function api
4. 最後的 api 像這樣 - https://path...../?currency=HKD



### 1 查看政府提供的幣值轉換資料

雖然政府的連結在這 - https://data.gov.tw/dataset/6708，不過匯率資料 api 在這 - https://portal.sw.nat.gov.tw/APGQ/GC331!downLoad?formBean.downLoadFile=CURRENT_JSON



### 2、利用 app script 把資料存進 google sheet


```js
function writeCurrentCustomsRatesToSheet() {
  const url = 'https://portal.sw.nat.gov.tw/APGQ/GC331!downLoad?formBean.downLoadFile=CURRENT_JSON';
  const response = UrlFetchApp.fetch(url);
  const data = JSON.parse(response.getContentText());

  // 確認資料結構正確
  if (!data || !data.items || data.items.length === 0) {
    Logger.log("⚠️ 沒有資料可寫入");
    return;
  }

  const spreadsheetId = '115E6zusM58GE-Tvb0aw6vuWMirlJlqdqY5pnuavG1oI';
  const sheetName = '匯率';
  const sheet = SpreadsheetApp.openById(spreadsheetId).getSheetByName(sheetName);
  if (!sheet) throw new Error(`❌ 找不到工作表 "${sheetName}"`);

  sheet.clear();
  SpreadsheetApp.flush();
  Utilities.sleep(500);

  // 寫入標題列
  const dateRange = `${data.start} ~ ${data.end}`;
  sheet.appendRow(['幣別', '買入匯率', '賣出匯率', '適用期間']);

  // 寫入每個幣別資料
  data.items.forEach(item => {
    sheet.appendRow([
      item.code,
      item.buyValue,
      item.sellValue,
      dateRange
    ]);
  });

  Logger.log(`✅ 共寫入 ${data.items.length} 筆匯率資料（期間：${dateRange}）`);
}
```


### 3、讀取 google sheet 的資料，寫一隻 cloud function api


```py
import functions_framework
from flask import jsonify, request, make_response
from google.oauth2.service_account import Credentials
from googleapiclient.discovery import build

# === 設定 ===
SPREADSHEET_ID = "115E6zusM58GE-Tvb0aw6vuWMirlJlqdqY5pnuavG1oI"
SHEET_NAME = "匯率"
SCOPES = ["https://www.googleapis.com/auth/spreadsheets.readonly"]

# === 授權 ===
creds = Credentials.from_service_account_file("credentials.json", scopes=SCOPES)
service = build("sheets", "v4", credentials=creds)

@functions_framework.http
def get_currency_rate(request):
    # ✅ 處理 CORS 預檢請求（OPTIONS）
    if request.method == "OPTIONS":
        return _build_cors_preflight_response()

    currency = request.args.get("currency", "").upper()
    if not currency:
        return _cors_response(jsonify({"error": "請提供 currency 參數"}), 400)

    try:
        range_name = f"{SHEET_NAME}!A2:D"
        result = service.spreadsheets().values().get(
            spreadsheetId=SPREADSHEET_ID,
            range=range_name
        ).execute()

        rows = result.get("values", [])

        for row in rows:
            if row[0] == currency:
                return _cors_response(jsonify({
                    "currency": row[0],
                    "buy": float(row[1]),
                    "sell": float(row[2]),
                    "range": row[3]
                }))

        return _cors_response(jsonify({"error": f"找不到幣別: {currency}"}), 404)

    except Exception as e:
        import traceback
        print("❌ 錯誤：", e)
        traceback.print_exc()
        return _cors_response(jsonify({"error": "內部錯誤"}), 500)

# === CORS 預檢（OPTIONS）處理 ===
def _build_cors_preflight_response():
    response = make_response("")
    response.headers.set("Access-Control-Allow-Origin", "*")
    response.headers.set("Access-Control-Allow-Methods", "GET, OPTIONS")
    response.headers.set("Access-Control-Allow-Headers", "Content-Type")
    response.status_code = 204
    return response

# === 加上 Access-Control-Allow-Origin 標頭的 JSON 回應 ===
def _cors_response(response, status=200):
    if not isinstance(response, str):
        response = make_response(response, status)
    else:
        response = make_response(response, status)

    response.headers.set("Access-Control-Allow-Origin", "*")
    return response

```

   
   
   
   
### 4. 使用該 api

可以依照想要轉換的幣值，在 `currency` 這個 param 改變你想要的 value
```md
https://waveshine-currency-exchange-993916594104.asia-east1.run.app/?currency=HKD
```



:::tip
這個專案有一個特性，就是我原本是用 app script 來寫第三步驟，但是因為 app script 沒辦法改動 `Access-Control-Allow-Origin` 也就是 CORS 的問題，
但是我需要在前端使用這個 api ，必須要打開他，因此改成用 cloud function 來建立 api
:::





