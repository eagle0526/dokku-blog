---
sidebar_position: 1
---

0、前言
------
:::tip
**寫此篇文章的動機**  
此檔案是來介紹如何 merged 兩個 dict 的教學，另外一種類型
:::



1、透過相同的 key，merged 兩個 dict
------

這一次第一個 dict 有兩層 dict，第二個 dict 要把它融進第二層裡面

* 下面是兩個 dict 的格式

```py
# 第一個 dict
dict1 = {
    '846': {'11/1-11/30': {'OP表_FB花費': 109628,
                           'OP表_GADS花費': 0,
                           'OP表_TIKTOK花費': 0,
                           'ec_id': '846',
                           '客戶名稱': '第一金_表單_家庭傳媒',
                           '實際廣告花費走期': '11/1-11/30',
                           '實際開發票金額(未稅)': 169600,
                           '當月預算(含服)': 180000,
                           '發票號碼': 'HF43181735',
                           '總預算(含服)': 180000,
                           '預估當月毛利': 61740}},
    '946': {'11/1~11/30': {'OP表_FB花費': 0,
                           'OP表_GADS花費': 0,
                           'OP表_TIKTOK花費': 0,
                           'ec_id': '946',
                           '客戶名稱': '上通電子',
                           '實際廣告花費走期': '11/1~11/30',
                           '實際開發票金額(未稅)': 19479,
                           '當月預算(含服)': 20000,
                           '發票號碼': 'HF43181652',
                           '總預算(含服)': 20000,
                           '預估當月毛利': 2530}}
}


# 第二個 dict
dict2 = {
    '9': {'Dashboard_FB花費': '11718.30',
          'Dashboard_GADS花費': '0.00',
          'Dashboard_Tiktok花費': '0.00'},
    '921': {'Dashboard_FB花費': '1382868.00',
            'Dashboard_GADS花費': '0.00',
            'Dashboard_Tiktok花費': '0.00'},
    '946': {'Dashboard_FB花費': '0.00',
            'Dashboard_GADS花費': '11299.74',
            'Dashboard_Tiktok花費': '0.00'}
}
```

* 來把兩個融合到一起

```py
# 先從 dict1 取出 ec_id 和 他的 value 值
for ec_id, dict1_details in dict1.items():
    # 用 dict1 的 ec_id 取得 dict2 是否有這個 ec_id 在 dict2 中
    dict2_entry = dict2.get(ec_id)

    # 再把 dict1 最內層的 dict 拆分成 period 和 data
    for period, data in dict1_details.items():
        # 如果 'dict2' 也有 'Dashboard_FB花費'，那就把該筆資料加進 dict1 中
        data["Dashboard_FB花費"] = dict2_entry.get("Dashboard_FB花費", "0.00")
```