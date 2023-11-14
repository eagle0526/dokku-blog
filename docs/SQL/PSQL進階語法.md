---
sidebar_position: 2
---

## Rank、Dense_Rank、Row_Number 用法與範例

* RANK： 分配排名，如果兩個值相同，則會有相同的排名，但下一個排名會跳過相應的數量。    
* DENSE_RANK： 分配排名，如果兩個值相同，則會有相同的排名，但下一個排名不會跳過相應的數量。    
* ROW_NUMBER： 為每一行分配唯一的整數值，而不考慮重複。    

### 建立範例資料
#### 新增 table

先新增一個 `table`，裡面有兩個欄位，一個是產品名稱，一個是產品價錢
```shell
test1234=# CREATE TABLE products (
             id SERIAL PRIMARY KEY,
             title VARCHAR,
             price INTEGER
           );
```


#### 新增資料

```shell
test1234=# INSERT INTO products (title, price)
VALUES
  ('One', 10),
  ('Two', 20),
  ('Three', 20),
  ('Four', 30);
```

輸入後，我們來印出來：
```shell
test1234=# select * from products;

------
 id | title | price 
----+-------+-------
  1 | One   |    10
  2 | Two   |    20
  3 | Three |    20
  4 | Four  |    30
(4 rows)
```


### Rank 

RANK 會一起和 OVER 一起使用：

```shell
test1234=# select price, RANK() OVER (ORDER BY price) AS rank
test1234-# FROM products;

------
 price | rank 
-------+------
    10 |    1
    20 |    2
    20 |    2
    30 |    4
(4 rows)
```


印出來後，會發現兩個相同價錢的產品，都有相同的 `2` 排名，並且下一個排名的產品，就會變成 `4`。


### Dense_Rank

```shell
test1234=# select price, DENSE_RANK() OVER (ORDER BY price) AS rank FROM products;

------
 price | rank 
-------+------
    10 |    1
    20 |    2
    20 |    2
    30 |    3
(4 rows)
```

使用 `Dense_Rank` 後，可以發現跟 `rank` 最大的差異，就是原本會跳號碼的 `30` 元產品，現在不會跳號碼了，就算有重複的產品，依然會照著數字排泄來。


### ROW_NUMBER

```shell
test1234=# select price, ROW_NUMBER() OVER (ORDER BY price) AS rank FROM products;

------
 price | rank 
-------+------
    10 |    1
    20 |    2
    20 |    3
    30 |    4
(4 rows)
```


使用 `ROW_NUMBER` 會發現，rank 的排名，就算有相同價錢的產品，依然不會重複。



