---
sidebar_position: 4
---

## @retry 介紹

今天來介紹一個可以讓我們的函示或方法重複使用、驗證的 `裝飾器 decorator`，他是用在實現重試機制，也就是當函示、方法出現錯誤時，可以自動重複執行，直到成功或是你限定的重試次數


### 實際舉個例子

現在我們實作一個簡單的函示，也就是重複呼叫函示三次，這個函示在做 - `得到隨便一個數字，如果 < 0.8，就印出錯誤，反之會印出成功`


1. 首先要先安裝 tenacity 這個 package

```shell
$ pip install tenacity
```

2. 再來引入程式後，把程式寫完

```py
from tenacity import retry, stop_after_attempt, wait_fixed, after_log
import logging

def log_attempt_number(retry_state):
    logging.warning(f"Retrying: {retry_state.attempt_number}...")

@retry(
    stop = stop_after_attempt(3),
    wait = wait_fixed(2),
    after= log_attempt_number
)
def do_something():
    print("Doing something....")
    import random
    if random.random() < 0.8:
        raise Exception("something wrong!!!")        
    else:
        print("Operation succeeded!")

```
執行這個程式檔的話，會印出類似下面的資訊：

如果今天觸發 exception，就會回去再執行一次 `do_something`，直到他成功，或者是到你設定錯誤的次數

```md
Doing something....
WARNING:root:Retrying: 1...
Doing something....
WARNING:root:Retrying: 2...
Doing something....
Operation succeeded!
```