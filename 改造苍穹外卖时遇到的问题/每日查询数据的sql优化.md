

## 改之前的问题
原来用 for 循环遍历 30 天，每天查 2 次数据库（总订单数 + 有效订单数）
= 60 次 SQL 查询

## 怎么发现的
看代码发现 for 循环里有 countAllOrders 和 getStatusAndVaildOrder 两次调用

## 怎么改的
写了一条 SQL，用 GROUP BY DATE() 按天分组，
用 COUNT(CASE WHEN status = 5 THEN 1 END) 条件统计有效订单
一条 SQL 同时返回每天的总数和有效数

## 改之后的效果
60 次 SQL → 1 次 SQL
Java 代码从 for 循环查库改为 Stream 在内存中组装

## 踩过的坑
- @Param 参数名和 XML 里的 #{xxx} 必须一致
- MySQL datetime 对应 LocalDateTime，不能用 LocalDate
- MyBatis resultType="map" 返回 Map<String, Object>，key 是列名字符串