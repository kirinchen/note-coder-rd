---
title: Pandas
tags: [database, data science, python, 量化交易]

---







# Dataframe

## read_csv to Dataframe
```python=
data_scv = pd.read_csv(filepath_or_buffer='./data/1701269629597.csv')
```

## 常用
```python=
data_scv.head() # 看前五行

data_scv.tail() # 看後五行

print(data_scv.dtypes) # 看所有欄位的型別

print(data_scv.deep_val) # 取其中一欄並回傳為 pandas.series

X_train = data_scv[['m5_20','m5_60']] # 取其中2欄並回傳為 pandas.Dataframe

data_scv.columns.values # 取得所有欄位名稱
data_scv[data_scv.columns.values[0:-1]] # 取0-最後一欄的 DataFrame
```

## group
```python=
  data_scv["group"] = pd.cut(data_scv.AD_CAT, np.arange(-11.0, 12.0, 1.0), right=False)
    
    df = pd.DataFrame({"value": np.random.randint(0, 100, 20)})
    df["group"] = pd.cut(df.value, range(0, 105, 10), right=False)
    df.head(10)
    print(df.head())    
```
output:
```shell=
   value     group
0     70  [70, 80)
1     19  [10, 20)
2     69  [60, 70)
3     43  [40, 50)
4      9   [0, 10)
```
### DataFrameGroupBy
```python=
group: DataFrameGroupBy = df.groupby(['group']) #
first_group:DataFrame = group.get_group((list(group.groups)[0])) # 取第一個 group
```