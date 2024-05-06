---
sidebar_position: 3
---

## Django 的 enum 使用


* 先說明一下 `enum` 的作用是什麼：     

列舉（Enum）在程式設計中的作用是定義一組命名的常量集合，使程式碼更具可讀性、可維護性和可擴展性。使用列舉可以將相關的常量分組在一起，並使用有意義的名稱來表示它們，從而提高程式碼的可讀性和可理解性。     
     
以下是列舉在程式設計中的一些常見設計：   
1. 定義常量：列舉允許您將一組相關的常量集中在一起，並為它們分配有意義的名稱。這樣可以使程式碼更易於理解和維護。
2. 提高可讀性：使用列舉可以使程式碼更具可讀性，因為它可以提供有意義的名稱來表示常量，而不是使用數字或字串。
3. 避免錯誤：列舉可以幫助避免拼寫錯誤或使用錯誤的常量值，因為它們提供了一組預先定義的合法值。
4. 代表狀態：列舉可以用來表示某些狀態或類型的可能值，從而使程式碼更具表達性。
5. 支持迭代和比較：列舉類型通常支持迭代和比較操作，使得在處理常量集合時更加方便和容易。
總的來說，列舉是一種有用的工具，它可以提高程式碼的可讀性、可維護性和可擴展性，並且可以幫助避免一些常見的錯



參考資料來源 : 
1. https://blog.louie.lu/2017/08/02/%E4%BD%A0%E6%89%80%E4%B8%8D%E7%9F%A5%E9%81%93%E7%9A%84-python-%E6%A8%99%E6%BA%96%E5%87%BD%E5%BC%8F%E5%BA%AB%E7%94%A8%E6%B3%95-07-enum/
2. https://stackoverflow.com/questions/29503339/how-to-get-all-values-from-python-enum-class

### cron-job-v2 


以下是之前主管在此專案上用到 `enum` 的地方，先來說明一下為什麼要像以下這樣設計：

1. `@classmethod` 是 `django` 的 `Decorator`，可以定義一個類別方法，簡單說就是可以讓 `BaseEnum` 變成一個類別方法，如果今天沒有加上 `@classmethod`，這個 class 是物件方法，下面會展示
2. 一開始先建立一些 `Enum` 未來可能會用到的方法，之後讓那些 `其他類別` 繼承 `BaseEnum`，ex. DatabaseNameEnum.get_values()，可以把那些資料都變成陣列，並且印出他的值，下面會展示


```python
from enum import Enum
from typing import List


class BaseEnum(Enum):
    @classmethod
    def has_value(cls, value):
        return value in cls._value2member_map_

    @classmethod
    def get_values(cls) -> List:
        return [enum_obj._value_ for enum_obj in cls._value2member_map_.values()]


class DatabaseNameEnum(BaseEnum):
    old_db_primary = 'old-db-primary'
    old_db_replica = 'old-db-replica'
    new_db_primary = 'new-db-primary'
    new_db_replica = 'new-db-replica'


class DatabaseLabelEnum(BaseEnum):
    old_db = 'old-db'
    new_db = 'new-db'
```



## 類別方法 vs 物件方法

* `cls` 代表是類別本身
```python
# 類別方法
class MyClass:
    @classmethod
    def my_class_method(cls, arg1, arg2):
        print(cls)
        print(arg1)
        print(arg2)

# 使用類別名稱調用類別方法
MyClass.my_class_method(10, 20)



# 物件方法
class OtherClass:
    def my_instance_method(self, arg1, arg2):
        print(self)     # self 將會是該類別的實例
        print(arg1)
        print(arg2)

obj = OtherClass()
obj.my_instance_method(30, 40)

```

## BaseEnum 的方法呼叫

```python
class BaseEnum(Enum):
    @classmethod
    def has_value(cls, value):
        return value in cls._value2member_map_

    @classmethod
    def get_values(cls) -> List:
        return [enum_obj._value_ for enum_obj in cls._value2member_map_.values()]


class DatabaseNameEnum(BaseEnum):
    old_db_primary = 'old-db-primary'
    old_db_replica = 'old-db-replica'
    new_db_primary = 'new-db-primary'
    new_db_replica = 'new-db-replica'


print(DatabaseNameEnum.has_value("old-db-replica"))     # True
print(DatabaseNameEnum.has_value("test"))               # False
print(DatabaseNameEnum.get_values())                    # ['old-db-primary', 'old-db-replica', 'new-db-primary', 'new-db-replica']
```



## 其他關於 Enum 的方法

以下是一些關於 `enum` 值取得的一些方法：

```python
class InputType(Enum):
    Track = 'track'
    Book = 'book'


print(InputType.Book == InputType.Track)       # False
print(InputType.Book == InputType.Book)        # True


print(InputType.Book.name)                     # Book
print(InputType.Book.value)                    # book

print(list(InputType))                         # [<InputType.Track: 'track'>, <InputType.Book: 'book'>]
print(InputType['Track'])                      # InputType.Track


print( e for e in InputType )                  # <generator object <genexpr> at 0x100f20040>
print( e.value for e in InputType )            # <generator object <genexpr> at 0x100f20040>
```

