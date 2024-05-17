---
sidebar_position: 2
---

0、前言
------
:::tip
**寫此篇文章的動機**  
這一篇是用爬蟲來抓 Pleague 球員的資料
:::


## request 爬蟲

```py
from typing import Optional
from fastapi import FastAPI

import requests
import bs4
import pandas as pd
import urllib.parse
import time

app = FastAPI() # 建立一個 Fast API application

def parse_data():
    res = requests.get(f"https://pleagueofficial.com/stat-player/2023-24/2#record")    
    html_format = bs4.BeautifulSoup(res.text, "html.parser")

    table_PL = html_format.findAll("table")[0]
    # 資料 title 
    table_title_PL = table_PL.findAll("thead")[0].findAll("tr")[0].findAll("th")    
    # 資料 info
    table_data_PL = table_PL.findAll("tbody")[0].findAll("tr")

    # title的資料放在此陣列裡面
    title_arr = []
    # 最後把全部資料放在陣列裡面
    player_arr = []

    for i in range(0, len(table_title_PL)):
        title_arr.append(table_title_PL[i].text)    

    for i in range (0,len(table_data_PL)):
        player_Name = table_data_PL[i].findAll("th")[0].text #球員姓名
        player_Number = table_data_PL[i].findAll("td")[0].text #球員號碼
        player_Team = table_data_PL[i].findAll("td")[1].text #球員隊伍
        player_Game = table_data_PL[i].findAll("td")[2].text #出賽次數
        player_Time = table_data_PL[i].findAll("td")[3].text #出賽時間

        player_2FG_Goal = table_data_PL[i].findAll("td")[4].text #兩分進球次數
        player_2FG_Times = table_data_PL[i].findAll("td")[5].text #兩分出手次數
        player_2FG_Rate = table_data_PL[i].findAll("td")[6].text #兩分命中率

        player_3FG_Goal = table_data_PL[i].findAll("td")[7].text #三分進球次數
        player_3FG_Times = table_data_PL[i].findAll("td")[8].text #三分出手次數
        player_3FG_Rate = table_data_PL[i].findAll("td")[9].text #三分命中率

        player_FT_Goal = table_data_PL[i].findAll("td")[10].text #罰球進球次數
        player_FT_Times = table_data_PL[i].findAll("td")[11].text #罰球出手次數
        player_FT_Rate = table_data_PL[i].findAll("td")[12].text #罰球命中率

        player_Score = table_data_PL[i].findAll("td")[13].text #得分
        player_OREB = table_data_PL[i].findAll("td")[14].text #進攻籃板
        player_DREB = table_data_PL[i].findAll("td")[15].text #防守籃板
        player_REB = table_data_PL[i].findAll("td")[16].text #總籃板
        player_AST = table_data_PL[i].findAll("td")[17].text #助攻
        player_STL = table_data_PL[i].findAll("td")[18].text #抄截
        player_BLK = table_data_PL[i].findAll("td")[19].text #阻攻
        player_TO = table_data_PL[i].findAll("td")[20].text #失誤
        player_PF = table_data_PL[i].findAll("td")[21].text #犯規
        
        # 現在要把所有的球員塞到一個陣列裡面，應該可以考慮直接用二維陣列，最外面是一個陣列，每個球員是另外一個陣列
        FullPlayer_PL = [
            player_Name,
            player_Number,
            player_Team,    
            player_Game,    
            player_Time,     
            player_2FG_Goal,     
            player_2FG_Times,
            player_2FG_Rate,
            player_3FG_Goal,     
            player_3FG_Times,
            player_3FG_Rate,
            player_FT_Goal,      
            player_FT_Times, 
            player_FT_Rate, 
            player_Score,   
            player_OREB,    
            player_DREB,    
            player_REB,     
            player_AST,     
            player_STL,     
            player_BLK,     
            player_TO,      
            player_PF,
        ]
        player_arr.append(FullPlayer_PL)
    
    return title_arr, player_arr
```


### 把球員資料印出來

假設今天我只想要印出 `富邦` 球員的資訊，可以照著以下的架構來印出來

```py
from typing import Optional
from fastapi import FastAPI

import requests
import bs4
import pandas as pd
import urllib.parse
import time

app = FastAPI() # 建立一個 Fast API application

def parse_data():
    # ...省略

@app.get("/") # 指定 api 路徑 (get方法)
def read_root():

    title_arr, player_arr = parse_data()

    print(title_arr)
    for player in player_arr:
        if "富邦" in player[2]:                    
            print(player)

    return {"good": player_arr[1]}    
```


### 用 queryString 來抓取不同的球隊資料

但是今天使用者不可能只想要單一球隊的資料，一定還會想要其他球隊的資訊，因此我們改成用 `queryString` 的方式讓使用輸入自己想要查詢的球隊：

但這邊有一個重點，就是今天想要在 `queryString` 輸入中文，記得要使用 `urllib` 這個插件，如果不解析的話，程式只會收到一大堆亂碼

```py
from typing import Optional
from fastapi import FastAPI

import requests
import bs4
import pandas as pd
import urllib.parse
import time

app = FastAPI() # 建立一個 Fast API application

def parse_data():
    # ...省略

@app.get("/") # 指定 api 路徑 (get方法)
def read_root():
    # ...省略

@app.get("/pleague/{team}") 
def pleague_team(team: str):
    # 解析中文
    decoded_team = urllib.parse.unquote(team)

    title_arr, player_arr = parse_data()    
    for player in player_arr:        
        if decoded_team in player[2]:
            print(player)

    return {"title_arr": title_arr}
```






















