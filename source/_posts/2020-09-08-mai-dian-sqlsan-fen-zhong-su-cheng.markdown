---
layout: post
title: "拉埋点数据SQL三分钟速成"
date: 2020-09-08 17:06:40 +0800
comments: true
categories: 
---


### 1. 基本选取

我们现在有一个表 user_action，有如下数据：

```sql
SELECT 
*
FROM user_action
;
```

| app_id | app_version |device_id| sys_ver | event_name |
|:------------- |:---------------:| -------------:| -------------:|-------------:|
| 21370000 | 8.15.2 | x000001| 13.3 | btn1_touched |
| 21370000 | 8.15.2 | x000001| 13.3 | btn1_touched |
| 21370000 | 8.15.2 | x000001| 13.3 | btn1_touched |
| 21370000 | 8.15.2 | x000002| 14.0 | btn2_touched |
| 21370000 | 8.15.2 | x000003| 14.0 | btn1_touched |
| 21370000 | 8.15.2 | x000003| 14.0 | btn2_touched |

app_id app_ver 表示 app 和其版本号  
device_id 为设备id  
sys_ver 为系统版本号  
event_name 为具体的事件名  

<!--more-->

### 2. 选取特定列 (SELECT 子句)

```
SELECT
app_version,
device_id,
sys_ver,
event_name
FROM user_action
;
```

| app_version |device_id| sys_ver | event_name |
|:------------- |:---------------:| -------------:| -------------:|-------------:|
| 8.15.2 | x000001| 13.3 | btn1_touched |
| 8.15.2 | x000001| 13.3 | btn1_touched |
| 8.15.2 | x000001| 13.3 | btn1_touched |
| 8.15.2 | x000002| 14.0 | btn2_touched |
| 8.15.2 | x000003| 14.0 | btn1_touched |
| 8.15.2 | x000003| 14.0 | btn2_touched |

### 3. 过滤条件 (WHERE 子句)
```
SELECT
app_version,
device_id,
sys_ver,
event_name
FROM user_action
WHERE
sys_ver = '13.3'
;
```

| app_version |device_id| sys_ver | event_name |
|:------------- |:---------------:| -------------:| -------------:|-------------:|
| 8.15.2 | x000001| 13.3 | btn1_touched |
| 8.15.2 | x000001| 13.3 | btn1_touched |
| 8.15.2 | x000001| 13.3 | btn1_touched |

WHERE 子句全部运算符：

```
=   等于
!=  不等于
<   小于
<=           小于等于
>            大于
>=           大于等于
BETWEEN      在两者之间
IS NULL      为空
IS NOT NULL  非空
LIKE         字符串包含
```

有些 DB 还支持：

```
<> 不等于
!< 不小于
!> 不大于
```
### 4. 组合条件 (WHERE ... AND/OR/IN/NOT IN ...)：

* 结合顺序：先算 AND 后算 OR
* AND 和 OR 运算符在左值满足的情况下不计算右值的行为随 DB 不同而不同，但是不关心中间过程的情况下，这个区别一般不会影响计算结果

#### 4.1 AND

```
SELECT
app_version,
device_id,
sys_ver,
event_name
FROM user_action
WHERE
sys_ver = '14.0'
AND 
event_name = 'btn2_touched'
;
```
| app_version |device_id| sys_ver | event_name |
|:------------- |:---------------:| -------------:| -------------:|-------------:|
| 8.15.2 | x000002| 14.0 | btn2_touched |
| 8.15.2 | x000003| 14.0 | btn2_touched |

#### 4.2 IN

```
SELECT
app_version,
device_id,
sys_ver,
event_name
FROM user_action
WHERE
sys_ver = '14.0'
AND 
event_name IN ['btn1_touched', 'btn2_touched']
;
```
| app_version |device_id| sys_ver | event_name |
|:------------- |:---------------:| -------------:| -------------:|-------------:|
| 8.15.2 | x000002| 14.0 | btn2_touched |
| 8.15.2 | x000003| 14.0 | btn1_touched |
| 8.15.2 | x000003| 14.0 | btn2_touched |

#### 4.3 NOT IN

```
SELECT
app_version,
device_id,
sys_ver,
event_name
FROM user_action
WHERE
sys_ver = '14.0'
AND 
event_name NOT IN ['btn2_touched']
;
```
| app_version |device_id| sys_ver | event_name |
|:------------- |:---------------:| -------------:| -------------:|-------------:|
| 8.15.2 | x000003| 14.0 | btn1_touched |

#### 4.4 NOT

```
SELECT
app_version,
device_id,
sys_ver,
event_name
FROM user_action
WHERE
NOT sys_ver = '14.0'
;
```
| app_version |device_id| sys_ver | event_name |
|:------------- |:---------------:| -------------:| -------------:|-------------:|
| 8.15.2 | x000001| 13.3 | btn1_touched |
| 8.15.2 | x000001| 13.3 | btn1_touched |
| 8.15.2 | x000001| 13.3 | btn1_touched |


### 5. 聚集函数 (COUNT,AVG,MAX,MIN,SUM) 与分组 (GROUP BY)

* SELECT 中未使用聚集函数的所有列都要写在 GROUP BY 子句里
* 如果 SELECT 的全部列都使用了聚集函数，可以不写 GROUP BY，但是输出报表时这种情况并不常见  
* 表内有的两行数据行A和行B，若行A和行B在 GROUP BY 子句指定的任意一列中不一致，则行A和行B会被归到结果集不同的两行中，不被聚集
* COUNT() 接受列名或者 *。传入列名时，返回列内不为 NULL 的总数量，传入 * 时，返回列内含 NULL 的总数量
* 比如性能耗时均值，可以用 AVG 比较方便的计算出来

例子1：  

```
SELECT
app_version,
device_id,
sys_ver,
event_name,
COUNT(*) as 样本量

FROM user_action

WHERE
sys_ver = '13.3'

GROUP BY
app_version,
device_id,
sys_ver,
event_name
;
```

| app_version |device_id| sys_ver | event_name | 样本量 |
|:------------- |:---------------:| -------------:| -------------:|-------------:|-------------:|
| 8.15.2 | x000001| 13.3 | btn1_touched | 3 |

例子2：表内有的两行数据行A和行B，若行A和行B在 GROUP BY 子句指定的任意一列中不一致，则行A和行B会被归到结果集不同的两行中，不被聚集：    

```
SELECT
app_version,
device_id,
sys_ver,
event_name,
COUNT(*) as 样本量

FROM user_action

WHERE
sys_ver = '14.0'

GROUP BY
app_version,
device_id,
sys_ver,
event_name
;
```

| app_version |device_id| sys_ver | event_name | 样本量 |
|:------------- |:---------------:| -------------:| -------------:|-------------:|-------------:|
| 8.15.2 | x000002| 14.0 | btn2_touched | 1 |
| 8.15.2 | x000003| 14.0 | btn1_touched | 1 |
| 8.15.2 | x000003| 14.0 | btn2_touched | 1 |

### 6. 排序(ORDER BY)

```
SELECT
app_version,
device_id,
sys_ver,
event_name,
COUNT(*) as 样本量

FROM user_action

WHERE
sys_ver = '14.0'

GROUP BY
app_version,
device_id,
sys_ver,
event_name

ORDER BY
event_name
;
```

| app_version |device_id| sys_ver | event_name | 样本量 |
|:------------- |:---------------:| -------------:| -------------:|-------------:|-------------:|
| 8.15.2 | x000003| 14.0 | btn1_touched | 1 |
| 8.15.2 | x000002| 14.0 | btn2_touched | 1 |
| 8.15.2 | x000003| 14.0 | btn2_touched | 1 |


### 7. 去除列内相同值的数据(DISTINCT)

#### 7.1 在 COUNT/AVG/SUM 内部使用

最常见的是配合 COUNT 使用。COUNT(DISTINCT user_id) 常用于计算 UV、活跃设备数:  

```
SELECT
COUNT(DISTINCT device_id) as 激活设备数
FROM user_action
;
```

| 激活设备数 |
|:------------- |
| 3 |

在 COUNT 内部使用，仅可以匹配 COUNT(列), 不能匹配 COUNT(*)。匹配结果会包含 NULL。  
DISTINCT 除了可以作用于 COUNT/AVG/SUM，也可以作用于 MIN/MAX，但是 MIN/MAX 加 DISTINCT 没啥区别。  

#### 7.2 作用于 SELECT 子句

作用于 SELECT 子句时，紧跟 SELECT 写，对所有列生效。不能对部分列生效：

例子1 单列：  

```
SELECT DISTINCT
app_version,
FROM user_action
;
```

| app_version |
|:------------- |
| 8.15.2 |


例子2 多列：  

```
SELECT DISTINCT
app_version,
device_id,
FROM user_action
;
```

| app_version |device_id|
|:------------- |:---------------:|
| 8.15.2 | x000001|
| 8.15.2 | x000002|
| 8.15.2 | x000003|


### 8. 限制结果集数量(LIMIT)

使用 LIMIT 一般有两种情况：
* 提升执行速度
* 有数据库读取条数权限控制，不允许读出更多条

LIMIT 仅对结果集行数限制，和中间计算过程无关：  

```
SELECT
app_version,
device_id,
sys_ver,
event_name,
COUNT(*) as 样本量

FROM user_action

WHERE
sys_ver = '13.3'

GROUP BY
app_version,
device_id,
sys_ver,
event_name

LIMIT 1
;
```

| app_version |device_id| sys_ver | event_name | 样本量 |
|:------------- |:---------------:| -------------:| -------------:|-------------:|-------------:|
| 8.15.2 | x000001| 13.3 | btn1_touched | 3 |


### 9. 索引

* 索引建在经常需要 WHERE 的列上。

* WHERE 中以下表达式会导致单次运算无法使用索引：  
	* WHERE 子句的查询条件里有 !=
	* WHERE 子句使用了 Mysql 函数的时候: WHERE left(name, 4) = 'xxx'
	* 使用 LIKE，匹配关键字带有前置通配符时，无法使用索引: 
		* WHERE name LIKE '%Alisa%' 无法使用
		* WHERE name LIKE 'Alisa%' 可以使用
		* WHERE name LIKE '%Alisa' 无法使用
	* WHERE 子句使用了 IS NULL 或者 IS NOT NULL
	* WHERE 子句条件中有 OR，即使其中有条件带索引也不会使用


