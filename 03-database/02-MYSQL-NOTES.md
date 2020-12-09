# MySQL Notes


### HEX

```sql
SELECT HEX('18620208080');        -- Output: 3133393231323038333232
SELECT HEX(18620208080);          -- Output: 455D9D3D0
SELECT UNHEX(0x455D9D3D0);        -- Output: Null
SELECT UNHEX('455D9D3D0');        -- Output: =��
SELECT CONV('455D9D3D0', 16, 10); -- Output: 18620208080
```


### UNIX_TIMESTAMP 2038年过期问题(等于0)

```sql
SELECT UNIX_TIMESTAMP('2120-01-01 00:00:00');
-- 替换为
SELECT TIMESTAMPDIFF(SECOND, '1970-01-01 08:00:00', '2120-01-01 00:00:00');
```


### BETWEEN AND 包含边界
 * `gmt_create` BETWEEN '2019-01-01 00:00:00' AND '2019-03-31 23:59:59'
 * 等价 `gmt_create` >= '2019-01-01 00:00:00' AND `created_datetime` <= '2019-03-31 23:59:59'


### DECIMAL(10, 2)
 * 全长为10位
 * 小数为2位，整数为 (10 - 2) = 8 位


### IN/NOT IN, EXISTS/NOT EXISTS的区别


### LENGTH(), CHAR_LENGTH(), VARCHAR(N)
 > 示例字段: varchar(9) 

```sql
SELECT str, CHAR_LENGTH(str), LENGTH(str) FROM tmp_v2;
'中华人民共和国万岁', 9, 27
```


### 表关联字段字符集不一致
 > 错误提示 

```
[Err] 1267 - Illegal mix of collations (utf8mb4_general_ci,IMPLICIT) and (utf8mb4_unicode_ci,IMPLICIT) for operation '='
```

 > 解决方法，CONVERT 字符集
```
ON CONVERT(a.store_code USING utf8mb4) COLLATE utf8mb4_unicode_ci = b.store_code
```


### datetime 字段，使用函数后全表扫描

```sql
EXPLAIN
SELECT * 
  FROM crm_order_info 
 WHERE DATE_FORMAT(gmt_create, '%Y-%m-%d') = DATE_SUB(CURDATE(), INTERVAL 1 DAY)
;

EXPLAIN
SELECT * 
  FROM crm_order_info 
 WHERE gmt_create BETWEEN CONCAT(DATE_SUB(CURDATE(), INTERVAL 1 DAY), ' 00:00:00') AND CONCAT(DATE_SUB(CURDATE(), INTERVAL 1 DAY), ' 23:59:59')
;
```


### [Err] 1690 - BIGINT UNSIGNED value is out of range in '(`crm_order_info`.`pay_amount` - `crm_order_info`.`refund_amount`)'
 > 解决方法

```sql
SET sql_mode = 'NO_UNSIGNED_SUBTRACTION';
```


### 查看Mysql慢日志配置信息

```sql
SHOW VARIABLES LIKE '%slow_query%';

Variable_name       Value
slow_query_log      ON
slow_query_log_file	/data/db/slow_query.log
```


### 中文排序

```sql
ORDER BY CONVERT(`nickname` USING gbk) COLLATE gbk_chinese_ci ASC
``` 


### GROUP_CONCAT 多字段排序

```sql
SELECT GROUP_CONCAT(dict_value, '=', dict_title ORDER BY CAST(`dict_value` AS SIGNED) ASC)
  FROM crm_dictionary 
 WHERE dict_name = 'resource'
;
``` 


### 位运算

```sql
--  1: 开启功能A
--  2: 开启功能B
--  4: 开启功能C
--  8: 开启功能D
-- 16: 开启功能E
-- 32: 开启功能F

SET @ENABLED_ITEM = 1;  
SELECT user_id, enabled_group 
  FROM crm_user_property 
 WHERE (BINARY(enabled_group) & BINARY(@ENABLED_ITEM) = @ENABLED_ITEM)
;
```


### Data truncation: Truncated incorrect DOUBLE value: '4 in look';
Error querying database.  Cause: com.mysql.jdbc.MysqlDataTruncation: Data truncation: Truncated incorrect DOUBLE value: '4 in look'
The error may exist in class path resource [ods/ODSMapper.xml]
Cause: com.mysql.jdbc.MysqlDataTruncation: Data truncation: Truncated incorrect DOUBLE value: '4 in look'
; Data truncation: Truncated incorrect DOUBLE value: '4 in look'; nested exception is com.mysql.jdbc.MysqlDataTruncation: Data truncation: Truncated incorrect DOUBLE value: '4 in look'

解决方法：
user_id 字段为varchar(32)类型，需先UPDATE 然后转换类型为bigint(20)
```sql
UPDATE crm_order_info AS o
   SET o.user_id = CAST(o.user_id AS SIGNED)
 WHERE CAST(o.user_id AS SIGNED) != mt.user_id
;
ALTER TABLE `crm_order_info` ALGORITHM=INPLACE, LOCK=NONE, 
MODIFY COLUMN `user_id`  bigint(20) NULL DEFAULT NULL COMMENT '用户ID';
```