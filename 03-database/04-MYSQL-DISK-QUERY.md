# Mysql查看库、表占用存储空间大小

### 1. 查看该数据库实例下所有库大小

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


### 2、查看该实例下各个库大小

```sql
SELECT table_schema, 
       SUM(data_length+index_length)/1024/1024 AS total_mb, 
       SUM(data_length)/1024/1024 AS data_mb, 
       SUM(index_length)/1024/1024 AS index_mb, 
       COUNT(*) AS tables, 
       CURDATE() AS today 
  FROM information_schema.tables 
 GROUP BY table_schema 
 ORDER BY 2 DESC 
; 
```


### 3、查看单个库的大小

```sql
SELECT CONCAT(ROUND(SUM(data_length)/1024/1024, 2),'mb') AS data_size,  
       CONCAT(ROUND(SUM(max_data_length)/1024/1024, 2),'mb') AS max_data_size,   
       CONCAT(ROUND(SUM(data_free)/1024/1024, 2),'mb') AS data_free,  
       CONCAT(ROUND(SUM(index_length)/1024/1024, 2),'mb') AS index_size 
  FROM information_schema.tables 
 WHERE table_schema = 'db-mall'
;
```


### 4、查看单个表的状态

```sql
SHOW TABLE STATUS FROM `db-mall` WHERE `name` = 'crm_order_info'
;
```


### 5、查看单库下所有表的状态

```sql
SELECT table_name, 
       (data_length/1024/1024) AS data_mb, 
       (index_length/1024/1024) AS index_mb, 
       ((data_length+index_length)/1024/1024) AS all_mb, table_rows 
  FROM information_schema.tables 
 WHERE table_schema = 'db-mall' 
 ORDER BY (data_length+index_length) DESC 
;
```