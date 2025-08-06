---
sidebar_position: 1
---

0、前言
------
:::tip
**寫此篇文章的動機**  
此檔案是來介紹如何 merged 兩個 dict 的教學
:::



1、透過相同的 key，merged 兩個 dict
------

現在有兩個不同內容的 dict，唯一相同的就是，他們的 key 值有些是一樣的，我要把相同的 key，不同的內容，融合到一起：

* 下面是兩個 dict 的格式

```py
# 第一個 dict
dict1 = {'225': {'客戶名稱': '寰宇迪士尼_數網業外', 'ec_id': '225', '發票號碼': '11月待開', '實際開發票金額(未稅)': 11, '實際廣告花費走期': '10/1~10/31', '總預算(含服)': 100000, '當月預算(含服)': 100000, 'OP表_FB花費': 64773, 'OP表_GADS花費': 0, 'OP表_TIKTOK花費': 0, '預估當月毛利': 14259}, '946': {'客戶名稱': '上通電子', 'ec_id': '946', '發票號碼': 'FG41999487', '實際開發票金額(未稅)': 19390, '實際廣告花費走期': '10/11-10/30', '總預算(含服)': 20000, '當月預算(含服)': 20000, 'OP表_FB花費': 0, 'OP表_GADS花費': 0, 'OP表_TIKTOK花費': 0, '預估當月毛利': 2530}} 

# 第二個 dict
dict2 = {'100': {'Dashboard_FB花費': '16509.00', 'Dashboard_GADS花費': '15315.52', 'Dashboard_Tiktok花費': '0.00'}, '1007': {'Dashboard_FB花費': '0.00', 'Dashboard_GADS花費': '0.00', 'Dashboard_Tiktok花費': '0.00'}, '1011': {'Dashboard_FB花費': '0.00', 'Dashboard_GADS花費': '0.00', 'Dashboard_Tiktok花費': '0.00'}, '1029': {'Dashboard_FB花費': '16499.00', 'Dashboard_GADS花費': '0.00', 'Dashboard_Tiktok花費': '0.00'}, '1064': {'Dashboard_FB花費': '151288.80', 'Dashboard_GADS花費': '16932.36', 'Dashboard_Tiktok花費': '0.00'}, '107': {'Dashboard_FB花費': '356253.00', 'Dashboard_GADS花費': '353944.11', 'Dashboard_Tiktok花費': '0.00'}}
```

* 來把兩個融合到一起

```py
for ec_id in dict1.items():

    # 用 dict1 的 ec_id 抓出第二個 dict 的資料
    dict2_entry = dict2.get("ec_id")

    # 有的話，就直接把 dict2 的資料，補進 dict1 裡面
    if dict2_entry:
        dict1["Dashboard_FB花費"] = dict2_entry.get("dashboard facebook 花費", "0.00")
        dict1["Dashboard_GADS花費"] = dict2_entry.get("dashboard GADS 花費", "0.00")
        dict1["Dashboard_Tiktok花費"] = dict2_entry.get("dashboard Tiktok 花費", "0.00")            
    else:
        # 若在第二段資料中找不到對應 ec_id，補上默認值
        dict1["dashboard facebook 花費"] = "0.00"
        dict1["dashboard GADS 花費"] = "0.00"
        dict1["dashboard Tiktok 花費"] = "0.00"
```