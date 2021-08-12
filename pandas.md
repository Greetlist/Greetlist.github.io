# 一些pandas 代码片段

## apply
```
df[['new_col1', 'new_col2']] = df[['calc1', 'calc2', 'calc3']].apply(lambda x: calc_func(x), axis=1)

def calc_func(args):
    # do something
    # finally return a Series
    return pd.Series([new_col1_data, new_col2_data])
```
