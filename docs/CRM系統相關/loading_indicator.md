---
sidebar_position: 1
---

0、前言
------
:::tip
**寫此篇文章的動機**  
此篇會介紹，如何做讀取 sheet 時的計時器
:::



1、計時器設定
------

* 先設定好要跑的時間

```py
def loading_indicator(stop_event):
    """顯示進度提示的函數"""
    counter = 1
    
    # 如果 stop_event.is_set() 這個是 false，下面就會一直執行，如果為 true 就會跳出
    while not stop_event.is_set():
        print(f"\rDashboard Data Loading {counter}...", end="", flush=True)     
        counter += 1
        time.sleep(1)
    
    print("\rLoading complete!")
```


2、讀取 sheet 時觸發計時器
------

```py
def filter_dashboard_data_in_date_range(start_date_str, end_date_str, ec_ids=None):
    # 新增 threading Event
    stop_event = threading.Event()
    # 觸發計時器，兩個參數 target = 剛剛設定的 fun， args 加入剛剛的變數
    loading_thread = threading.Thread(target=loading_indicator, args=(stop_event, ))
    # 讓計時器開始計算
    loading_thread.start()    

    # 在 try 運作的期間， threading 就會一直去運作 loading_indicator，因此會印出時間
    try:        
        start_date = datetime.strptime(start_date_str, "%Y-%m-%d")
        end_date = datetime.strptime(end_date_str, "%Y-%m-%d")
        dashboard_sheet_data = get_google_sheet_data()
        rows = dashboard_sheet_data[1:]

        filtered_data = [
            row for row in rows if start_date <= datetime.strptime(row[0], "%Y-%m-%d") <= end_date
        ]

        if ec_ids:
            ec_column_index = 4
            filtered_data = [row for row in filtered_data if row[ec_column_index] in ec_ids]

    # try 裡面的 func 跑完後，就會進到這邊
    finally:
        # 把 stop_event 參數轉成 true，停止 loading_indicator 運作
        stop_event.set()
        # 停止 thread 運作
        loading_thread.join()
    return filtered_data
```



3、其他方式了解 thread 運作
-----

```py
def boss(event):
    print('[B] I am boss')
    sleep(5)

    print('[B] All workers, work now')
    event.set()
    sleep(20)

    print('[B] All workers, take a break now')
    event.clear

def worker(event, worker_name):
    print('[W] I am worker', worker_name, ', I am waiting for the boss')
    event_is_set = event.wait()

    while event.is_set():
        color = choice(['red', 'black', 'green'])
        print('[W]', worker_name, 'choices', color)
        sleep(5)
        

event = threading.Event()

boss_thread = threading.Thread(name='boss', target=boss, args=(event, ))
worker_thread_1 = threading.Thread(name='worker_John', target=worker, args=(event, 'John'))
worker_thread_2 = threading.Thread(name='worker_Kate', target=worker, args=(event, 'Kate'))

boss_thread.start()
worker_thread_1.start()
worker_thread_2.start()
```




可以參考這一篇文章了解 thread 的運作：https://myapollo.com.tw/blog/python-event-objects/