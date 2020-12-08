# Mysql 分区滑动归档


# 零、表分区

### 0.1. 分区简介
 > 分区是将一个表分解为多个更小、更可管理的部分。就应用访问而言，只有一个表，但在物理上可能由数十个物理分区组成。

### 0.2. 分区类型
 * RANGE分区：连续区间，eg: 按时间范围年、月分区
 * LIST分区：离散枚举，eg: 按手机号码前三位对运营商进行分区
 * OTHER：留待深入

### 0.3. 分区约束
 > 分区要求分区字段必须是主键或者唯一索引的一部分，即主键、唯一索引均需要增加分区字段。
   比如 crm_order_info 需要用 gmt_create 作为分区字段，那么主键，以及所有唯一索引，均需要增加 gmt_create 字段。

### 0.4. 分区交换
 * 两张表必须具有完全相同的结构，但是必须一为分区表，一为普通空表；
   * so: 对于表结构修改，需要同步修改归档表。
 * 分区表与分区表之间无法直接交换分区，需要通过普通表搭桥中转；
 * 以下交换SQL可将2018年01月的区间数据，在两张表之前来回倒腾；

```sql
-- 分区表在前，普通空表在后
ALTER TABLE crm_order_info EXCHANGE PARTITION op201801 WITH TABLE crm_order_info_archive_201801;
```

### 0.5. 常用操作

```sql
-- 创建分区：普通表转为分区表
ALTER TABLE `crm_order_info` PARTITION BY RANGE COLUMNS(`gmt_create`) (
    PARTITION op201700 VALUES LESS THAN ('2018-01-01'), -- 2017年 + 2017年以前的数据分区
    PARTITION op201801 VALUES LESS THAN ('2018-02-01'), -- 2018年1月的数据分区
    PARTITION op201802 VALUES LESS THAN ('2018-03-01'),
    ...
    PARTITION op202007 VALUES LESS THAN ('2020-08-01'), -- 当前时间是 2020年8月
    PARTITION op000000 VALUES LESS THAN maxvalue
);

-- 反向操作：分区表转为普通表
-- ALTER TABLE crm_order_info REMOVE PARTITIONING;

-- 重新组织分区：合并分区
ALTER TABLE `crm_order_info` REORGANIZE PARTITION op201801,op201802,op201803,op201804,op201805,op201806,op201807,op201808,op201809,op201810,op201811,op201812 INTO (
    PARTITION op201800 VALUES LESS THAN ('2019-01-01')
);

-- 重新组织分区：拆分分区
ALTER TABLE `crm_order_info` REORGANIZE PARTITION op000000 INTO (
    PARTITION op202007 VALUES LESS THAN ('2020-08-01'),
    PARTITION op000000 VALUES LESS THAN maxvalue
);

-- 删除分区
-- ALTER TABLE crm_order_info DROP PARTITION op201700;

-- 查询分区信息
SELECT TABLE_SCHEMA, TABLE_NAME, PARTITION_NAME, TABLE_ROWS
  FROM information_schema.`PARTITIONS` 
 WHERE TABLE_NAME = 'crm_order_info' 
   AND TABLE_SCHEMA = 'db-mall' 
 ORDER BY partition_description DESC 
;

-- 按分区查询
SELECT *
  FROM crm_order_info PARTITION(op201803)
 LIMIT 100
;
```



# 一、准备工作
### 1.1. 原始表：选定分区字段以及主键修改

```sql
-- ALGORITHM=INPLACE, LOCK=NONE 是 RDS for MySQL 5.6 支持的 ONLINE DDL 特性。
-- 如果要修改主键，删除主键和添加主键建议放在一条语句中，以便充分利用 5.6 版本的 Online DDL 特性。

ALTER TABLE `crm_order_info` ALGORITHM=INPLACE, LOCK=NONE, 
DROP PRIMARY KEY, ADD PRIMARY KEY (`id`, `gmt_create`),
DROP INDEX `uk_order_no`, ADD UNIQUE INDEX `uk_order_no` (`order_no`, `gmt_create`) USING BTREE;
```

### 1.2. 创建归档表模板

```sql
CREATE TABLE crm_order_info_archive LIKE crm_order_info;
```

### 1.3. 原始表分区

```sql
ALTER TABLE `crm_order_info` PARTITION BY RANGE COLUMNS(`gmt_create`) (
    PARTITION op201700 VALUES LESS THAN ('2018-01-01'), -- 2017年 + 2017年以前的数据分区
    PARTITION op201801 VALUES LESS THAN ('2018-02-01'), -- 2018年1月的数据分区
    PARTITION op201802 VALUES LESS THAN ('2018-03-01'),
    PARTITION op201803 VALUES LESS THAN ('2018-04-01'),
    PARTITION op201804 VALUES LESS THAN ('2018-05-01'),
    PARTITION op201805 VALUES LESS THAN ('2018-06-01'),
    PARTITION op201806 VALUES LESS THAN ('2018-07-01'),
    PARTITION op201807 VALUES LESS THAN ('2018-08-01'),
    PARTITION op201808 VALUES LESS THAN ('2018-09-01'),
    PARTITION op201809 VALUES LESS THAN ('2018-10-01'),
    PARTITION op201810 VALUES LESS THAN ('2018-11-01'),
    PARTITION op201811 VALUES LESS THAN ('2018-12-01'),
    PARTITION op201812 VALUES LESS THAN ('2019-01-01'),
    PARTITION op201901 VALUES LESS THAN ('2019-02-01'),
    PARTITION op201902 VALUES LESS THAN ('2019-03-01'),
    PARTITION op201903 VALUES LESS THAN ('2019-04-01'),
    PARTITION op201904 VALUES LESS THAN ('2019-05-01'),
    PARTITION op201905 VALUES LESS THAN ('2019-06-01'),
    PARTITION op201906 VALUES LESS THAN ('2019-07-01'),
    PARTITION op201907 VALUES LESS THAN ('2019-08-01'),
    PARTITION op201908 VALUES LESS THAN ('2019-09-01'),
    PARTITION op201909 VALUES LESS THAN ('2019-10-01'),
    PARTITION op201910 VALUES LESS THAN ('2019-11-01'),
    PARTITION op201911 VALUES LESS THAN ('2019-12-01'),
    PARTITION op201912 VALUES LESS THAN ('2020-01-01'),
    PARTITION op202001 VALUES LESS THAN ('2020-02-01'),
    PARTITION op202002 VALUES LESS THAN ('2020-03-01'),
    PARTITION op202003 VALUES LESS THAN ('2020-04-01'),
    PARTITION op202004 VALUES LESS THAN ('2020-05-01'),
    PARTITION op202005 VALUES LESS THAN ('2020-06-01'),
    PARTITION op202006 VALUES LESS THAN ('2020-07-01'),
    PARTITION op000000 VALUES LESS THAN maxvalue
);
```

### 1.4. 2017年 + 2017年以前的数据分区迁移

```sql
-- 创建2017年归档表
CREATE TABLE crm_order_info_archive_2017 LIKE crm_order_info_archive;

-- 普通表与分区表之间，交换分区就搞定了
ALTER TABLE crm_order_info EXCHANGE PARTITION op201700 WITH TABLE crm_order_info_archive_2017;
```

# 二、按月归档迁移
 > 2018年数据分区迁移
 > 当前时间是 2020年8月，业务需要保留两年的热数据，2018年及以后将存在部分月份存留，部分月份归档的情况。
 > 处理完归档部分，剩下将交给存储过程，每月自动完成滑动归档

```sql
-- STEP01: 创建2018年归档表
CREATE TABLE crm_order_info_archive_2018 LIKE crm_order_info_archive;

-- STEP02: 创建2018年归档表分区
ALTER TABLE `crm_order_info_archive_2018` PARTITION BY RANGE COLUMNS(`gmt_create`) (
    PARTITION op201801 VALUES LESS THAN ('2018-02-01'), 
    PARTITION op201802 VALUES LESS THAN ('2018-03-01'),
    ...
    PARTITION op201812 VALUES LESS THAN ('2019-01-01')
);

-- STEP03: 创建普通中转表，201808 留给存储过程测试
CREATE TABLE crm_order_info_archive_201801 LIKE crm_order_info_archive;
CREATE TABLE crm_order_info_archive_201802 LIKE crm_order_info_archive;
...
CREATE TABLE crm_order_info_archive_201807 LIKE crm_order_info_archive;

-- STEP04: 交换分区 => 月中转表
ALTER TABLE crm_order_info EXCHANGE PARTITION op201801 WITH TABLE crm_order_info_archive_201801;
ALTER TABLE crm_order_info EXCHANGE PARTITION op201802 WITH TABLE crm_order_info_archive_201802;
...
ALTER TABLE crm_order_info EXCHANGE PARTITION op201807 WITH TABLE crm_order_info_archive_201807;

-- STEP05: 月中转表 => 年归档表
ALTER TABLE crm_order_info_archive_2018 EXCHANGE PARTITION op201801 WITH TABLE crm_order_info_archive_201801;
ALTER TABLE crm_order_info_archive_2018 EXCHANGE PARTITION op201802 WITH TABLE crm_order_info_archive_201802;
...
ALTER TABLE crm_order_info_archive_2018 EXCHANGE PARTITION op201807 WITH TABLE crm_order_info_archive_201807;

-- STEP06: 删除月中转表
DROP TABLE crm_order_info_archive_201801;
DROP TABLE crm_order_info_archive_201802;
...
DROP TABLE crm_order_info_archive_201807;
```
注：没有删除 crm_order_info 中已归档的分区，是为了方便必要时，可以快速还原数据。



# 三、存储过程
### 用于分区维护的存储过程

```sql
CREATE PROCEDURE `proc_partition_crm_order_info`()
BEGIN

    -- Description: 滑动归档存储过程
    -- Author: ShawnPanda
    -- Update: 2020-08-21
 
    -- @param inTableSchema 归档数据库
    -- @param inTableName   归档数据表
    -- @param inPartPrefix  分区前缀
    -- @param inPartFiled   分区字段
    -- @param inAliveMonths 保留月份数
    -- @param outMsg 输出信息

    DECLARE outMsg VARCHAR(255) DEFAULT NULL;
    CALL proc_partition_common('db-mall', 'crm_order_info', 'op', 'gmt_create', 24, outMsg);
    SELECT outMsg;
 
END;



-- 年归档表初始化
-- @param inTableSchema 归档数据库
-- @param inTableName 归档数据表
-- @param inPartPrefix 分区前缀
-- @param inPartFiled 分区字段
-- @param inArchiveYear 归档年份
CREATE PROCEDURE `proc_partition_year_archive_table_init`(IN `inTableSchema` varchar(32), IN `inTableName` varchar(32), IN `inPartPrefix` varchar(32), IN `inPartFiled` varchar(32), IN `inArchiveYear` varchar(4))
BEGIN

    -- Description: 年归档表初始化
    -- Author: ShawnPanda
    -- Update: 2020-09-15

    DECLARE vArchiveYearMaxDay VARCHAR(10) DEFAULT NULL;
    DECLARE vArchiveYearTable  VARCHAR(64) DEFAULT NULL;
    DECLARE vCurrentSql      VARCHAR(2048) DEFAULT NULL;

    SELECT CONCAT((inArchiveYear + 1), '-01-01') INTO vArchiveYearMaxDay;

    -- STEP-A1: 检查年归档表是否存在
    SELECT TABLE_NAME INTO vArchiveYearTable 
      FROM information_schema.TABLES 
     WHERE TABLE_SCHEMA = inTableSchema 
       AND TABLE_NAME = CONCAT(inTableName, '_archive_', vArchiveYear) 
    ;

    -- STEP-A2: 不存在 => 创建
    IF vArchiveYearTable IS NULL THEN 
        -- STEP-B1: 创建年归档表
        SELECT CONCAT('CREATE TABLE ', inTableSchema, '.', inTableName, '_archive_', inArchiveYear, ' LIKE ', inTableName, '; ') INTO vCurrentSql;
        SET @vCurrentSql = vCurrentSql;
        PREPARE STMT FROM @vCurrentSql;
        EXECUTE STMT;
        DEALLOCATE PREPARE STMT;

        -- STEP-B2: 清除分区
        SELECT CONCAT('ALTER TABLE ', inTableSchema, '.', inTableName, '_archive_', inArchiveYear, ' REMOVE PARTITIONING; ') INTO vCurrentSql;
        SET @vCurrentSql = vCurrentSql;
        PREPARE STMT FROM @vCurrentSql;
        EXECUTE STMT;
        DEALLOCATE PREPARE STMT;

        -- STEP-B3: 建立分区
        SELECT CONCAT(
            'ALTER TABLE ', inTableSchema, '.', inTableName, '_archive_', inArchiveYear, ' PARTITION BY RANGE COLUMNS(', inPartFiled, ') (',
            '    PARTITION ', inPartPrefix, inArchiveYear, '01 VALUES LESS THAN (\'', inArchiveYear, '-02-01\'),',
            '    PARTITION ', inPartPrefix, inArchiveYear, '02 VALUES LESS THAN (\'', inArchiveYear, '-03-01\'),',
            '    PARTITION ', inPartPrefix, inArchiveYear, '03 VALUES LESS THAN (\'', inArchiveYear, '-04-01\'),',
            '    PARTITION ', inPartPrefix, inArchiveYear, '04 VALUES LESS THAN (\'', inArchiveYear, '-05-01\'),',
            '    PARTITION ', inPartPrefix, inArchiveYear, '05 VALUES LESS THAN (\'', inArchiveYear, '-06-01\'),',
            '    PARTITION ', inPartPrefix, inArchiveYear, '06 VALUES LESS THAN (\'', inArchiveYear, '-07-01\'),',
            '    PARTITION ', inPartPrefix, inArchiveYear, '07 VALUES LESS THAN (\'', inArchiveYear, '-08-01\'),',
            '    PARTITION ', inPartPrefix, inArchiveYear, '08 VALUES LESS THAN (\'', inArchiveYear, '-09-01\'),',
            '    PARTITION ', inPartPrefix, inArchiveYear, '09 VALUES LESS THAN (\'', inArchiveYear, '-10-01\'),',
            '    PARTITION ', inPartPrefix, inArchiveYear, '10 VALUES LESS THAN (\'', inArchiveYear, '-11-01\'),',
            '    PARTITION ', inPartPrefix, inArchiveYear, '11 VALUES LESS THAN (\'', inArchiveYear, '-12-01\'),',
            '    PARTITION ', inPartPrefix, inArchiveYear, '12 VALUES LESS THAN (\'', vArchiveYearMaxDay, '\')',
            ');') INTO vCurrentSql;
        SET @vCurrentSql = vCurrentSql;
        PREPARE STMT FROM @vCurrentSql;
        EXECUTE STMT;
        DEALLOCATE PREPARE STMT;
    END IF;
END;



-- 滑动归档公共存储过程
-- 归档最久的一个月份分区，新建上个月分区，总体保留最近 24 个月的数据。
-- @param inTableSchema 归档数据库
-- @param inTableName 归档数据表
-- @param inPartPrefix 分区前缀
-- @param inPartFiled 分区字段
-- @param inAliveMonths 保留月份数
-- @param outMsg 输出信息
CREATE PROCEDURE `proc_partition_common`(IN `inTableSchema` varchar(32), IN `inTableName` varchar(32), IN `inPartPrefix` varchar(32), IN `inPartFiled` varchar(32), IN `inAliveMonths` tinyint(2), OUT `outMsg` varchar(255))
LABPPC:BEGIN

    -- Description: 滑动归档公共存储过程
    -- Author: ShawnPanda
    -- Update: 2020-09-15

    DECLARE vArchiveDatetime       VARCHAR(20) DEFAULT NULL;
    DECLARE vArchiveYear           VARCHAR(10) DEFAULT NULL;
    DECLARE vArchiveYearMaxDay     VARCHAR(10) DEFAULT NULL;
    DECLARE vArchiveYearPartition  VARCHAR(10) DEFAULT NULL;
    DECLARE vArchiveMonth          VARCHAR(10) DEFAULT NULL;
    DECLARE vArchiveMonthMaxDay    VARCHAR(10) DEFAULT NULL;
    DECLARE vArchiveMonthPartition VARCHAR(10) DEFAULT NULL;
    DECLARE vArchiveMonthTable     VARCHAR(64) DEFAULT NULL;
    DECLARE vNewYearDay            VARCHAR(10) DEFAULT NULL;
    DECLARE vNewYearPartition      VARCHAR(10) DEFAULT NULL;
    DECLARE vLastPartition         VARCHAR(10) DEFAULT NULL;
    DECLARE vFirstPartition        VARCHAR(10) DEFAULT NULL;
    DECLARE vCurrentSql          VARCHAR(2048) DEFAULT NULL;

    -- 判断保留月份数
    -- 目前仅支持保留 3, 6, 12, 18, 24, 36 个月的数据
    IF inAliveMonths NOT IN (3, 6, 12, 18, 24, 36) THEN
        SELECT "EXIT: PROCEDURE OF WRONG INPUT ALIVE MONTHS, ONLY SUPPORT IN (3, 6, 12, 18, 24, 36)!" INTO outMsg;
        LEAVE LABPPC;
    END IF;

    SELECT DATE_SUB(NOW(), INTERVAL inAliveMonths MONTH) INTO vArchiveDatetime;
    SELECT YEAR(vArchiveDatetime)                        INTO vArchiveYear;
    SELECT CONCAT((vArchiveYear + 1), '-01-01')          INTO vArchiveYearMaxDay;
    SELECT DATE_FORMAT(vArchiveDatetime, '%Y%m') - 1     INTO vArchiveMonth;
    SELECT DATE_FORMAT(vArchiveDatetime, '%Y-%m-01')     INTO vArchiveMonthMaxDay;
    SELECT CONCAT(inPartPrefix, vArchiveMonth)           INTO vArchiveMonthPartition;
    SELECT CONCAT(inPartPrefix, vArchiveYear, '99')      INTO vArchiveYearPartition;
    SELECT CONCAT(inPartPrefix, YEAR(NOW()), '99')       INTO vNewYearPartition;
    SELECT DATE_FORMAT(NOW(), '%Y-01-01')                INTO vNewYearDay;

    -- STEP-A1: 检查归档分区是否存在
    SELECT PARTITION_NAME INTO vLastPartition 
      FROM information_schema.PARTITIONS 
     WHERE TABLE_SCHEMA = inTableSchema 
       AND TABLE_NAME = inTableName 
       AND PARTITION_NAME = vArchiveMonthPartition 
    ;

    -- STEP-A2: 不存在 => 创建归档分区
    IF vLastPartition IS NULL THEN 
        IF vArchiveYear < YEAR(NOW()) THEN
            SELECT CONCAT('ALTER TABLE ', inTableSchema, '.', inTableName, ' REORGANIZE PARTITION ', vArchiveYearPartition, ' INTO (', 
                              'PARTITION ', vArchiveMonthPartition, ' VALUES LESS THAN (''', vArchiveMonthMaxDay, '''), ', 
                              'PARTITION ', vArchiveYearPartition, ' VALUES LESS THAN (''', vArchiveYearMaxDay, ''') ', 
                          ');') INTO vCurrentSql;
        ELSE
            SELECT CONCAT('ALTER TABLE ', inTableSchema, '.', inTableName, ' REORGANIZE PARTITION ', vArchiveYearPartition, ' INTO (', 
                              'PARTITION ', vArchiveMonthPartition, ' VALUES LESS THAN (''', vArchiveMonthMaxDay, '''), ', 
                              'PARTITION ', vArchiveYearPartition, ' VALUES LESS THAN (maxvalue) ', 
                          ');') INTO vCurrentSql;
        END IF;

        SET @vCurrentSql = vCurrentSql;
        PREPARE STMT FROM @vCurrentSql;
        EXECUTE STMT;
        DEALLOCATE PREPARE STMT;
    END IF;

    -- STEP-B1: 检查归档分区是否需要归档(有数据)
    SET vLastPartition = NULL;
    SELECT PARTITION_NAME INTO vLastPartition 
      FROM information_schema.PARTITIONS
     WHERE TABLE_SCHEMA = inTableSchema
       AND TABLE_NAME = inTableName
       AND PARTITION_NAME = vArchiveMonthPartition 
       AND TABLE_ROWS > 0 
    ;

    -- STEP-B2: 有数据 => 执行归档流程
    IF vLastPartition IS NOT NULL THEN
        -- STEP-C1: 年归档表、分区
        CALL proc_partition_year_archive_table_init(inTableSchema, inTableName, inPartPrefix, inPartFiled, vArchiveYear);

        -- STEP-C2: 创建月中转表
        SELECT CONCAT(inTableSchema, '.', inTableName, '_archive_', vArchiveMonth) INTO vArchiveMonthTable;
        SELECT CONCAT('CREATE TABLE ', vArchiveMonthTable, ' LIKE ', inTableSchema, '.', inTableName, '_archive;') INTO vCurrentSql;
        SET @vCurrentSql = vCurrentSql;
        PREPARE STMT FROM @vCurrentSql;
        EXECUTE STMT;
        DEALLOCATE PREPARE STMT;

        -- STEP-C3: 交换分区 => 月中转表
        SELECT CONCAT('ALTER TABLE ', inTableSchema, '.', inTableName, ' EXCHANGE PARTITION ', vArchiveMonthPartition, 
                      ' WITH TABLE ', vArchiveMonthTable, ';') INTO vCurrentSql;
        SET @vCurrentSql = vCurrentSql;
        PREPARE STMT FROM @vCurrentSql;
        EXECUTE STMT;
        DEALLOCATE PREPARE STMT;

        -- STEP-C4: 月中转表 => 年归档表
        SELECT CONCAT('ALTER TABLE ', inTableSchema, '.', inTableName, '_archive_', vArchiveYear, ' EXCHANGE PARTITION ', vArchiveMonthPartition, 
                      ' WITH TABLE ', vArchiveMonthTable, ';') INTO vCurrentSql;
        SET @vCurrentSql = vCurrentSql;
        PREPARE STMT FROM @vCurrentSql;
        EXECUTE STMT;
        DEALLOCATE PREPARE STMT;

        -- STEP-C5: 删除月中转表
        SELECT CONCAT('DROP TABLE ', vArchiveMonthTable, ';') INTO vCurrentSql;
        SET @vCurrentSql = vCurrentSql;
        PREPARE STMT FROM @vCurrentSql;
        EXECUTE STMT;
        DEALLOCATE PREPARE STMT;
    END IF;

    -- STEP-D1. 获取最前分区
    SELECT PARTITION_NAME INTO vFirstPartition 
      FROM information_schema.PARTITIONS 
     WHERE TABLE_SCHEMA = inTableSchema 
       AND TABLE_NAME = inTableName 
       AND PARTITION_DESCRIPTION = 'maxvalue' 
     ORDER BY PARTITION_DESCRIPTION DESC 
     LIMIT 1
    ;

    -- STEP-D2. 创建新年分区
    IF vNewYearPartition != vFirstPartition THEN
        SELECT CONCAT('ALTER TABLE ', inTableSchema, '.', inTableName, ' REORGANIZE PARTITION ', vFirstPartition, ' INTO (', 
                          'PARTITION ', vFirstPartition, ' VALUES LESS THAN (''', vNewYearDay,'''), ', 
                          'PARTITION ', vNewYearPartition, ' VALUES LESS THAN (maxvalue) ', 
                      ');') INTO vCurrentSql;
        SET @vCurrentSql = vCurrentSql;
        PREPARE STMT FROM @vCurrentSql;
        EXECUTE STMT;
        DEALLOCATE PREPARE STMT;
    END IF;

END;



-- 快速归档
-- @param inTableSchema 归档数据库
-- @param inTableName 归档数据表
CREATE PROCEDURE `proc_fast_archive_table`(IN `inTableSchema` varchar(32), IN `inTableName` varchar(32))
BEGIN

    -- Description: 表快速备份
    -- Author: ShawnPanda
    -- Update: 2020-08-26

    DECLARE vCurrentDatetime VARCHAR(16)  DEFAULT NULL;
    DECLARE vCurrentSql      VARCHAR(200) DEFAULT NULL;
    DECLARE vLastInsertId    BIGINT(20)   DEFAULT 0;

    -- 1a. 归档表后缀 (202008251720 精确到分，防止异端频繁操作)
    -- 1b. 获取自增
    SELECT DATE_FORMAT(NOW(), '%Y%m%d%H%i') INTO vCurrentDatetime;
    SELECT CASE WHEN AUTO_INCREMENT IS NULL THEN 0 
           ELSE (AUTO_INCREMENT + 100) 
           END INTO vLastInsertId
      FROM information_schema.TABLES
     WHERE TABLE_SCHEMA = inTableSchema
       AND TABLE_NAME = inTableName
    ;

    -- 2. 原表 => 归档表
    SELECT CONCAT('ALTER TABLE ', inTableSchema, '.', inTableName, ' RENAME TO ', inTableSchema, '.', inTableName, '_', vCurrentDatetime, ';') INTO vCurrentSql;
    SET @vCurrentSql = vCurrentSql;
    PREPARE STMT FROM @vCurrentSql;
    EXECUTE STMT;
    DEALLOCATE PREPARE STMT;

    -- 3. 创建新表
    SELECT CONCAT('CREATE TABLE ', inTableSchema, '.', inTableName, ' LIKE ', inTableSchema, '.', inTableName, '_', vCurrentDatetime, ';') INTO vCurrentSql;
    SET @vCurrentSql = vCurrentSql;
    PREPARE STMT FROM @vCurrentSql;
    EXECUTE STMT;
    DEALLOCATE PREPARE STMT;

    -- 4. 设置新表自增
    IF vLastInsertId > 0 THEN
        SELECT CONCAT('ALTER TABLE ', inTableSchema, '.', inTableName, ' AUTO_INCREMENT = ', vLastInsertId, ';') INTO vCurrentSql;
        SET @vCurrentSql = vCurrentSql;
        PREPARE STMT FROM @vCurrentSql;
        EXECUTE STMT;
        DEALLOCATE PREPARE STMT;
    END IF;

END;

```

# 四、每月调用存储过程
XXL-JOB 调度执行



# 五、参考
RDS for Mysql  通过分区归档历史数据 (只有分区，没有归档)
mysql分区交换exchange partition (交换分区)


