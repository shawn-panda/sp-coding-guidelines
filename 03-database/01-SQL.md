# SQL一致性


### 一、约定 
 * 关键字大写：SELECT/FROM/JOIN/ON/WHERE/AS 等；
 * 关键字对齐：SELECT/FROM/LEFT/ON/WHERE/AND 靠右对齐；
 * 字段数控制：每行 5 个字段，便于数数；
 * 别名用缩写：不要使用a/b/c，使用常见约定；
   * eg: tbl_user_info AS u, tbl_order_info AS o
 * 表关联限制：最多不超过3张表关联；
 * 使用LIMIT：默认最大取数为 1000 条，防止应用 OOM；

### 二、示例

```sql
SELECT o.user_id, o.order_amount, o.pay_amount, o.pay_datetime, o.order_status,
       g.goods_id, g.goods_name, g.goods_price, g.goods_color, g.googs_weight, 
       u.user_name, u.user_gender, u.user_status, u.user_birthday
  FROM tbl_order_info AS o 
  LEFT JOIN tbl_user_info AS u 
    ON o.member_id = u.member_id 
  LEFT JOIN tbl_goods_info AS g 
    ON o.goods_id = g.goods_id 
 WHERE o.pay_datetime BETWEEN '2020-10-20 00:00:00' AND '2020-10-21 23:59:59'
   AND u.user_gender = 'FEMALE'
 LIMIT 0, 1000 
;
```

```sql
SELECT CONCAT(TABLE_SCHEMA, '.', TABLE_NAME) AS `TableName`,
       ROUND(SUM(DATA_LENGTH)/1024/1024/1024, 2) AS `TotalDataSize(GB)`,
       ROUND(SUM(INDEX_LENGTH)/1024/1024/1024, 2) AS `TotalIndexSize(GB)`,
       ROUND(SUM(DATA_LENGTH + INDEX_LENGTH)/1024/1024/1024, 2) AS `TotalUsedSize(GB)` 
  FROM information_schema.TABLES
 GROUP BY `TableName`
HAVING `TotalUsedSize(GB)` > 1
 ORDER BY `TotalUsedSize(GB)` DESC 
; 
```