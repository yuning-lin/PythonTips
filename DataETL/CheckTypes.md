## 使用套件
```python
import numpy as np
import pandas as pd
import math
```

## 空值們
### nan
```python
pd.isna(np.nan)
np.isnan(np.nan)
math.isnan(np.nan)

## 刪除欄位 col 中元素為 nan 的整列資料
df.dropna(subset=['col'])
## 同直行全為空值則刪除該行
df.dropna(axis=1, how='all')
## 列出欄位名稱為 col 中元素為 nan 的資料
df[df['col'].isna()]
df[df['col'].isnull()]
df[pd.isna(df['col'])]
## 列出欄位名稱為 col 中元素不為 nan 的資料
df[df['col'].notna()]
df[~df['col'].isna()]
df[df['col'].notnull()]
df[pd.notna(df['col'])]
```

### None
```python
is None
is not None
isinstance(None, type(None))
```

### NAT
```python
np.isnat(np.datetime64("NaT"))
np.isnat(np.datetime64('nat'))
np.isnat(np.datetime64('nAt'))

## nat 藉由轉換成 object，可用 isna() 辨別
df = df.astype(object).mask(df.isna(), np.nan)
```

## 指定資料格式
### 確認是否為某種格式
```python
isinstance(x, datetime) # datetime, int, str, float
```
### 確認是否為整數
```python
# 法一
def is_integer_number(x):
    if isinstance(x, int):
        return True
    if isinstance(x, float):
        return x.is_integer()
    return False

# 法二
def is_integer_number(x):
    if float(x)==int(x):
        return True
    else:
        return False

is_integer_number(2.3)
is_integer_number(2.0)
is_integer_number(2)
```
### 確認哪些欄位為數值
* [數值判斷效率：The benchmark results](https://stackoverflow.com/questions/354038/how-do-i-check-if-a-string-represents-a-number-float-or-int)
```python
num_cols = df.applymap(lambda x: str(x).replace('.', '', 1).isdigit()).all(axis=0)
lst_num_cols = num_cols[num_cols].index.tolist()
```
### 格式轉換
```python
## 將 dataframe 所有為 float 的欄位轉換成 int
print(df.dtypes)
float_df = df.select_dtypes(include=['float64'])
# df.select_dtypes(exclude = ['object'])
# df.select_dtypes(include= np.number)
# df.select_dtypes('number')
float_col = list(float_df.columns)
df[float_col] = df[float_col].applymap(np.int64)
```
