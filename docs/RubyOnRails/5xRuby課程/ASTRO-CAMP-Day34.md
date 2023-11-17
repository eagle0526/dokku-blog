---
title: ROR/Ruby
sidebar_label: "DAY34. [ASTROCamp] Ruby-12"
description: Ruby
last_update:
  date: 2022-11-28
keywords:
  - 5xRuby
  - Ruby
sidebar_position: 35
---



1、docker是什麼
------

:::tip
**WIKI：**
Docker是一個開放原始碼的開放平臺軟體，用於開發應用、交付（shipping）應用、執行應用。
Docker允許使用者將基礎設施（Infrastructure）中的應用單獨分割出來，形成更小的顆粒（容器，
從而提高交付軟體的速度。      
:::


不過看wiki覺得他好像在講文言文，我們來舉個例子好了！  

### 1-1、docker舉個例子  
如果今天你是用windows的人，通常是一個作業系統(windows)裡面，再裝一個作業系統(WSL)，然後再安裝ROR，這樣一層裝一層的通常會有點慢，所以有些人會直接裝Linux  
```md
> windows <-- WSL <-- Ruby/Rails
>             Linux
```

#### 1-1-1、為啥會提到windows的WSL呢？    
因為docker就是在windows裡面一層，提供了一個空間，你可以在裡面裝A系統、B系統、C系統。docker就是包住WSL的那一個空間    

#### 1-1-2、如果上面還聽不懂，我們用輪船來舉例
今天有一艘貨輪，上面載了很多貨物，這艘貨輪就是docker，上面的一堆貨物，就是系統，又可以稱為container  
Ex. 貨物可以是Linux、windows、x資料庫、y資料庫...之類的  
     
所以今天只要在電腦上安裝docker，你就可以安裝各式各樣的作業系統，不再僅限於WSL，因此我們可以利用docker讓大家的作業系統差不多(不同的環境裝差不多類似版本的軟體)    



:::tip
如果以後上班第一天，首要任務就是把環境裝起來  
要怎麼裝呢？docker有兩個設定檔案  
(1)dockerfile   
(2)docker-compose.yml   
如果公司有給你這兩個檔案，只要把這個執行下去，你的環境就完成了  
:::



:::info
使用dockerfile，就可以讓大家環境都一樣  
:::



### 1-2、安裝docker

1、M1、M2晶片下載Apple Chip  
2、比較舊的mac安裝intel Chip     
[docker官網連結](https://www.docker.com/)    

```md
> docker 載好後，可以直接跑某個印象檔(docker run)

> Docker
> Image(read only)
> container
> container = Image.new

> 因為Image不能被修改，所以我今天要用的話，要生出一個新的實體出來(container)
> 這個container就是剛剛說到的貨櫃(container)，也就是他可以是系統 or 資料庫
```



### 1-3、nginx
可以到docker.hub的網站，來下載一堆東西   
     
我們今天先用docker來安裝nginx    
```md
> docker run nginx
```

:::tip
win -> port 80     
WSL(port3000)  
這邊要先另外提個東西，一般來說windows外面port是80，WSL是連不到的(裡面假設是3000)   
外面是無法直接貫通到WSL的，所以如果今天想要讓WSL相連，可以下下面的指令     
Ps. 外面裡面是啥意思呢？就是一開始提到的，windows裡面會在包一層WSL，windows無法直接灌通WSL     
:::

下面語法的意思是，我想要用docker去跑nginx的伺服器，然後裡面是80，那個-p就是port，4567是自己設定的，你可以亂打，他就是外面的通道  
```md
> docker run -p 4567:80 nginx

> 這樣打後，就可以跑localhost:4567這個網址了
```

所有網站的預設伺服器是80，只要沒打的，最後面都是:80
```md
> google為例
> google.com:80

> ps. 443是https
```


:::danger
如果之後要讓網址上線，要開通道給別人進入  
ex. docker run -p 正式網址:80   
因為所有通道都是封閉的，一定要開個管道讓外面連到裡面，別人才能看到  
:::


### 1-4、docker run mysql
為啥剛個要講一大圈的docker呢？因為今天就是要下載MySQL，使用docker就可以直接把一大包東西載下來，不過因為老師有幫我們做好一包檔案，我們直接用老師的檔案執行就好    
     
接下來就是去下載，跑[MySQLWorkbench](https://dev.mysql.com/downloads/workbench/)     
:::tip
Docker這個技能，如果會的話，以後蠻吃香的  
:::


下載好老師給的檔案後，打下面的指令
```md
> ./run
```

會建議選擇一些檔案，直接選擇3，把所有檔案載下來，再來把重新跑./run，選擇3，就完成    
接下來到Workbench，點選首頁中有一個+號，並看老師檔案中的readme，密碼是helloworld，正確之後就可以進入     



2、MySQL資料庫
------
筆記做到12:00，SQL的基本上都是流水帳的狀態，有空再回來補


### 2-1、資料庫語法
```md
> create database hi; -> 建立資料庫    
> drop database hi; -> 刪除資料庫  
```

:::tip
SQL跟excel有什麼不一樣??    
sql可以做關聯，excel做關聯很痛苦  
:::


### 2-2、SQL是什麼？
SQL(Structured Query Language，結構化查詢語言)是一種特定目的程式語言  
用於管理關聯式資料庫管理系統（RDBMS），或在關係流資料管理系統（RDSMS）中進行流處理。  


### 2-3、關聯式資料庫(RDBMS)
關聯式資料庫是建立在關聯模型基礎上的資料庫，藉助於集合代數等數學概念和方法來處理資料庫中的資料。  
現實世界中的各種實體以及實體之間的各種聯絡均用關聯模型來表示。  

:::tip
**各式各樣的關聯式資料庫**     
Oracle(甲骨文-超貴)、MSSQL(微軟的)、MySQL免費、PostgreSQL免費、SQLite免費  
:::


### 2-4、資料型態
(1) 文字 - CHAR/VARCHAR/TEXT     
(2) 數字 - INTEGER/DECIMAL   
(3) 日期 - DATE/DATETIME     
(4) 二進位 - BLOB    


### 2-5、建立資料表 - 英雄資料表
```md
> id          INT           Auto Increment  流水編號
> name        VARCHAR(100)  NOT NULL        名字 - 必填
> gender      CHAR(1)             
> age         INT
> hero_level  CHAR(1)       NOT NULL
> hero_rank   INT
> description TEXT
```


### 2-6、建立資料表 - 反派資料表

SQL語法大小寫有差嗎？  
create database hello -> 這樣可以  
CREATE DATABASE hello -> 這樣也可以  

不過建議填寫的時候使用大小   
   
INT vs INTEGER   
這兩個是一樣的   

CHAR(10) vs VARCHAR(10)  
這兩個是不一樣的   
   
當你使用CHAR(10)   
我跟你要10個格子 -> 假設我要塞hello，把5個字塞進去後，還有五個空格，不過五個空格還是會在那邊   
也就是10個個子，你一開始要幾格，就是幾格   
   
當你使用VARCHAR(10)  
我跟你要10個格子 -> 假設我要塞hello，把5個字塞進去後，還有五個空格，剩下五個空格會被記錄成1個  
也就是5+1(五個格子 + 紀錄剩下格子)   
   
那一定VARCHAR比較好嗎？不一定，如果今天身分證字號一定要10格，那最後VARCHAR就變成10+1   
那不就比CHAR的10格還大了嗎   


資料表追加欄位   
ALTER TABLE heros  
ADD COLUMN super_power VARCHAR(100)   
   
PS. 斷行對資料庫是沒有差的   


刪除欄位
ALTER TABLE heros        -> 這一行的意思是，我要改變指定的table 
DROP COLUMN super_power   




### DDL/DML/DQL

DDL  
Data Definition Language 資料定義語言  
create   
drop   
alter  


DML  
Data Manipulate Language 資料操作語言  
insert   
delete   
update   


DQL  
Data Query Language 資料查詢語言   
select   
   
   
   
INSERT INTO heros  
(name, gender)   
values   
("奇遇", "M")  
   
Ps. 文字型態需要用引號包起來   
數字型態則不需要引號   
   
省略欄位   
如果今天INSERT後面都省略，不過values都有寫   
這樣代表全部資料都要塞進去   



寫入的欄位順序一定要照順序嗎   
看狀況，如果今天是用省略語法，那就要照順序   
而且如果你今天只增加特定的值，你的值就要跟前面的欄位對上   




沒寫入的欄位的預設值是什麼   
如果沒有寫預設值，大家都不一樣，有可能是空欄位，也有可能是null   


多餘的單引號   
   
('Speed-o\'-Sound Sonic') ->增加一個反斜線，才可以把東西塞進去   



查詢資料(R)  

查詢所有英雄的資料   
```sql
SELECT * FROM awesome_db.heros  
```


或者是把資料表設定為預設值
save as default schema

設定為預設值後，
SELECT * FROM heros

這在SQL是註解的意思
-- hero.all 


過濾資料 - 我只想找S級的英雄，就可以用where來找
SELECT * FROM awesome_db.heroes 
where hero_level = "s";


在rails 硬寫SQL
```md
> User.find_by_sql("select * from users")
```


一個條件不夠，可以再加and
```sql
SELECT * FROM awesome_db.heroes 
where hero_level = "s"
AND gender = "f";
```
rails 的寫法
```md
> Hero.where(hero_level: 's', gender: 'F')
```


只撈部分欄位(不用*號，用指定欄位)
```sql
SELECT name, age FROM awesome_db.heroes 
where hero_level = "s"
AND gender = "f";
```
rails 寫法
```md
> Hero.select(:name, :age).where(hero_level: 's', gender: 'F')
```

為什麼只要部分欄位？select * 不好嗎？
因為如果table中有一個圖片欄位，那用*拉出來，不就一次拉一大包出來嗎！！
主要目的就是要省記憶體，一次拉特定欄位出來就好




下午的課



哪些人沒填年紀的，使用is可以怕null抓出來
```sql
SELECT age FROM awesome_db.heroes 
where age is null;
```
rails 寫法
```md
> Hero.select(:age).where(age: nil)
```

#### 搜尋名字裡背心關鍵字的英雄
這個%的意思是，文字的前面或後面還有字元，相似字的查找是LIKE
```sql
SELECT * FROM awesome_db.heroes 
where name LIKE '%背心%';
```

rails 寫法
硬塞一個字串進去
```md
> Hero.where("name like '%背心%'")
```


#### 找指定的年齡區間
```sql
SELECT * FROM awesome_db.heroes 
where age between 10 and 25 ;
```
rails 寫法
硬塞一個字串進去
```md
> Hero.where("age >= 10").where("age <= 25")
> Hero.where("age >= 10 and age <= 25")

> User.where(id: 10..25)  -> 這個可以做出between的效果
```


#### 列出所有S級和Ａ級hero

```sql
> 使用or寫法
SELECT * FROM awesome_db.heroes 
where hero_level = "s" or hero_level = "a" ;

> 使用in的寫法
SELECT * FROM awesome_db.heroes 
where hero_level in ('s', 'a') ;
```

rails 寫法，強塞字串
```md
> Hero.where("hero_level = "s" or hero_level = "a"")

> Hero.where(hero_level: "s").or(Hero.where(hero_level: "a"))
> Hero.where(hero_level: "s").or(Hero.where(hero_level: "a"))

> 使用in的寫法 - 給他一個中括號
> Hero.where(hero_level: ["s", "a"])
```



#### 列出不是S的英雄，使用 `not` 、 `用!=` 、 `用<>`
```sql
SELECT * FROM awesome_db.heroes 
where not hero_level = "s"  ;
```

用`!=`寫法
```sql
SELECT * FROM awesome_db.heroes 
where hero_level != "s"  ;
```

用`<>`寫法
```sql
SELECT * FROM awesome_db.heroes 
where hero_level <> "s"  ;
```


rails 寫法，強塞字串
```md
> Hero.where("hero_level != "s") 

> 使用not，這不用塞字串的方法
> Hero.where.not(hero_level: "s") 
```

#### 列出所有不是S也不是Ａ

```sql
SELECT * FROM awesome_db.heroes 
where hero_level != "s" and hero_level != "a";
```

使用in的寫法
```sql
SELECT * FROM awesome_db.heroes 
where hero_level not in ("s", "a");
```

rails 寫法
```md
> 使用not，這不用塞字串的方法
> Hero.where.not(hero_level: ["s", "a"]) 
```


列出所有A級男性英雄的姓名和年齡
```sql
SELECT name, age FROM awesome_db.heroes 
where gender = "M" and hero_level = "A" ;
```

rails 寫法
```md
> Hero.where.not(hero_level: ["s", "a"]) 
```



### Update

當你id = 25，把他的age改成10，英雄階級改成"A"
```sql
Update heros
SET age = 10, hero_level = "A"
WHERE id = 25
```

rails 寫法
```md
> Hero.find(25).update(age: 10)
```


#### 所有人都老一歲

```sql
Update heroes
SET age = age + 1
```

#### 請把id=35的英雄等級，改為B級101位
```sql
UPDATE heroes
set hero_level = "B", hero_rank = 101
where id = 35;
```


### Delete

DELETE FROM heroes
where hero_level = 'c'

rails 寫法，用迴圈跑
```md
> Hero.where(hero_level: "c").each do |h|
> h.destroy
```

用delete_all
```md
> Hero.delete_all(hero_level: "c")
```

> --  
> **delete VS destroy**   
>   
> destroy會走CallBack流程  
> delete不會走，會直接刪掉  
> --  



### 更進階的查詢

#### 計算count
可以算出總共多少筆數
```sql
SELECT count(*) FROM awesome_db.heroes 
where hero_level = "A" ;
```

rails 寫法，使用count
```md
> Hero.where(hero_level: "A").count
```


#### 加總SUM
```sql
SELECT sum(age) FROM awesome_db.heroes 
where hero_level = "A"
```

rails 寫法
```md
> Hero.where(hero_level: "A").sum(:age)
```

#### 平均average
```sql
SELECT AVG(age) FROM awesome_db.heroes 
where hero_level = "A" ;
```

rails 寫法
```md
> Hero.where(hero_level: "A").avg(:age)
```



#### 最大值max、最小值min
```sql
SELECT max(age) FROM awesome_db.heroes
```

rails 寫法
```md
> Hero.where(hero_level: "A").maximum(:age)
```



### 分組

以年齡的總和當作分類，算出每一個英雄等級的年齡總和
```sql
SELECT hero_level, sum(age) FROM awesome_db.heroes 
group by hero_level;
```


rails 寫法
```md
> 還沒寫
```


### 挑出不同的 distinct
15:58
這個是錯的，要再練習一下
SELECT distinct hero_level FROM awesome_db.monsters;



### 排序order

SQL正續寫法
```sql
SELECT * FROM awesome_db.heroes 
where hero_level = "s"
order by hero_rank;
```

rails 寫法 - desc倒敘寫法
```md
> Hero.where(hero_level: "S").order(hero_rank: :desc)
```


### 限量limit
```sql
SELECT *
FORM heroes
where hero_level = "s"
order by hero_rank
limit 3
```


rails 寫法 - 只拿前面3筆
```md
> Hero.where(hero_level: "S").order(hero_rank: :desc).limit(3)
```




### 更多表格的查詢
16:15

#### 主鍵
每一張table都會有一張primary key

:::tip
SQL中的setNull的意思是，當你源頭的id被改了，table把原本的關聯設定空值  
SQL中的cascade的意思是，當你源頭的id被改了，關聯就跟著改  
SQL中的restrict的意思是，當你源頭的id被改時，表會直接跟你說不准改  
:::


#### 外部鍵
foreign  key
參照到另一個資料表的主鍵
其實是資料表的某一個藍位
用來月任該資料表的紀錄跟被對照的表格的資料是對照起來的
跟主鍵不同，外部鍵不需要獨一無二


外部鍵的限制



子查詢 - 兩段查詢

下面做的是，我找出所有被埼玉打倒的怪物
```sql
SELECT * FROM monsters
where kill_by = (
		select id 
        from heroes 
        where name 
        LIKE '%埼%');
```


被埼玉且傑諾斯一起打倒的怪物
```sql
SELECT * FROM monsters
where kill_by in (
		select id 
        from heroes 
        where name in ('%埼%', '傑諾斯'));
```







### join 交集

join的目的，做跨資料表的查詢
假設有第三表，裡面只有放一堆的id
那要怎麼把兩邊的id轉成中文，就是分別join兩張表(所以我們這邊在操作三張表)
join兩張表後，另外把兩張表的id用=抓下來，就可以把數字轉成名字了

把對戰紀錄表中的兩邊id，都轉成英雄、怪物的中文名稱
```sql
select h.name, m.name
from battle_histories as b
join heroes as h
join monsters as m
on b.hero_id = h.id and b.monster_id = m.id
```

Inner Join - 交集
Left Join - 以左邊的表為主
Right Join - 以右邊的表為主




inner join - 把兩張表都有的留下來
```sql
select * from t1
inner join t2
on t1.username = t2.name
```



左邊為主 - Left Join
```sql
select * from t1
left join t2
on t1.username = t2.name
```


右邊為主 - Right Join
```sql
select * from t1
right join t2
on t1.username = t2.name
```


