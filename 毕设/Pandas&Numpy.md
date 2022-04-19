# Pandas

## 拼接`concat`

```python
concat([df1, df2, df3, ...], axis=0) #按行拼接
concat([df1, df2, df3, ...], axis=1) #按列拼接

由于index导致拼接错误的时候，可以通过reset_index(drop=True),重设并删除原索引的方式解决该问题。
```



# Numpy