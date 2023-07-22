# 窗口函数

窗口函数的一般范式

```hive
Function() Over (Partition By Column1，Column2，Order By Column3)
```

窗口函数又分为以下三类： 聚合型窗口函数、分析型窗口函数、取值型窗口函数

## 聚合类的窗口函数

聚合型 SUM(),MIN(),MAX(),AVG(),COUNT() 

需要注意的是累计去重的demo的时候
count(distinct xxx) 在窗口函数中是不运行被使用的， 可以用size(collect_set over(partition by order by))来代替

## 分析型窗口函数

分析型 rank() row_number() dense_rank(),cume_dist(),percent_rank(),ntile()
row_number 产生连续的序列号
rank 产生排序相同的序列号会跳过
dense_rank 不跳过下一序号

## 取值型窗口函数

LAG() LEAD() FISRTVALUE() LASTVALUE()
LAG 使用延迟数据

