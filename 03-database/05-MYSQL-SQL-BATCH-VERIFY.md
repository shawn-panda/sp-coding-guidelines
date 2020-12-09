# 批量验证SQL
 * 主要使用游标 + EXPLAIN


### 存储过程

```sql
CREATE PROCEDURE `proc_sql_batch_verify`(OUT outErrLine int(10))
BEGIN

    -- Author: ShawnPanda
    -- Update: 2020-11-02

    -- 创建接收游标数据的变量
    DECLARE vCurrentId  INT(10) DEFAULT 0;
    DECLARE vCurrentSql VARCHAR(5000);
    DECLARE vCountValue INT(10) DEFAULT 0;

    -- 创建结束标志变量
    DECLARE done INT DEFAULT FALSE;

    -- 创建游标
    DECLARE cur CURSOR FOR 
    SELECT id, CONCAT('EXPLAIN ', sql_clause) AS sqlExplain
      FROM crm_sql_info
    ;

    -- 指定游标循环结束时的返回值
    DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = TRUE;

    -- 打开游标
    OPEN cur;

    -- 开始循环游标里的数据
    read_loop:LOOP

        -- 根据游标当前指向的一条数据
        FETCH cur INTO vCurrentId, vCurrentSql;

        -- 判断游标的循环是否结束，是则跳出游标循环
        IF done THEN 
            LEAVE read_loop;
        END IF;

        SET @vCurrentSql = vCurrentSql;
        PREPARE STMT FROM @vCurrentSql;
        EXECUTE STMT;
        DEALLOCATE PREPARE STMT;

        SET vCountValue = vCountValue + 1;

    -- 结束游标循环
    END LOOP;

    -- 关闭游标
    CLOSE cur;

    SELECT vCountValue INTO outErrLine;
    
END
```

### 验证执行

```sql
SET @outErrLine = 0;
CALL proc_sql_batch_verify(@outErrLine);
SELECT @outErrLine;
```