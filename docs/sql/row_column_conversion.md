# 行列转换

## lateral view 配合UTDF函数使用，把一行查分为多行的数据

```hive
select …… from tabelA lateral view UDTF(xxx) 视图名 as a,b,c
```

### 与explode连用

```hive
-- 假设我们有一张表pageAds，它有两列数据
-- 第一列是pageid(string类型)，第二列是adid_list(Array<int>类型)，即用逗号分隔的广告ID集合：
pageid     adid_list
"front_page"  [1, 2, 3]
"contact_page"  [3, 4, 5]
 
-- 要统计所有广告ID在所有页面中出现的次数。

-- 首先分拆广告ID：

SELECT 
pageid, adid 
FROM pageAds .
LATERAL VIEW explode(adid_list) adTable AS adid;
    
-- 执行结果如下：pageid(string类型),adid(int类型)
pageid   adid
"front_page" 1
"front_page" 2
"front_page" 3
"contact_page" 3
"contact_page" 4
"contact_page" 5

-- 接下来就是一个聚合的统计：

SELECT 
adid, count(1) 
FROM pageAds 
LATERAL VIEW explode(adid_list) adTable AS adid
GROUP BY adid;

-- 执行结果如下：

adid count(1)
1   1
2   1
3   2
4   1
5	
		1

```

### 与parse_url_tuple连用

```hive
--准备数据:vim /export/datas/lateral.txt
1	http://facebook.com/path/p1.php?query=1
2	http://www.baidu.com/news/index.jsp?uuid=frank
3	http://www.jd.com/index?source=baidu

--创建表
create table tb_url(
id int,
url string
) row format delimited fields terminated by '\t';
--加载数据
load data local inpath '/export/datas/lateral.txt' into table tb_url;
--使用UDTF解析
SELECT parse_url_tuple(url, 'HOST') from tb_url;

--使用UDTF+lateral view
select 
  a.*,
  b.host,
  b.path
from 
  tb_url a 
  lateral view parse_url_tuple(url, 'HOST',"PATH") b as host,path;

--对比
SELECT id,parse_url_tuple(url, 'HOST') from tb_url;--失败，UDTF函数不能与字段连用
select id, a.* from tb_url lateral view parse_url_tuple(url, 'HOST','PATH') a as host,path;

```

## 二、explode

分类：UDTF
功能：函数可以将一个array或者map展开
explode(array)：
将array列表里的每个元素生成一行
explode(map)：
每一对元素作为一行，key为一列，value为一列