# 数据表设计一致性(MySQL)


### 一、数据表
 * 字符集: utf8mb4_general_ci；
 * 表引擎: InnoDB；
 * 表命名: 系统_业务模块_功能，小写字母，下划线分割，不超过 32 字符；
   * 示例: crm_user_info / crm_payment_config
 * 表注释: 标注表名，创建人，创建时间，修改时间；
 * 表必备三字段: uuid(雪花id), gmt_create, gmt_modified；
 * 是否删除: is_delete （数据库 tinyint 0/1 => POJO 布尔型且不加is前缀）；
 * 字段数量: 单表小于 32 个字段，小字段放在主表，大字段( varchar(255) / text)放在扩展表；
 * 分表: 单表超过 2000 万行或 增量 1000 万行/年，才推荐进行分区 > 分表 > 分库；



### 二、字段
 * 风格: 小写字母，下划线分割，不超过 32 字符；
 * 类型: 合适的字符类型以及存储长度；
   * 示例: 2.1.常用字段设计
 * 时间: 统一使用datetime，废弃 timestamp；
 * 注释: 属性定义 + 枚举类json
   * 示例1: 性别 { 0:未知; 1:男性; 2:女性; 3:女变男; ... }
   * 示例2: 状态 { 0:初始化; 1:开始; 2:进行中; 3:暂停; ... 9:结束; }

#### 2.1.常用字段设计

| 字段 | 命名 | 类型 | 定义 |
|-----|-----|-----|-----|
| 性别 | gender | unsigned tinyint | { 0:未知; 1:男性; 2:女性; 3:女变男; ... } |
| 年龄 | age | unsigned tinyint | 0 - 255 |
| 状态 | status | unsigned tinyint | { 0:初始化; 1:开始; 2:进行中; 3:暂停; ... 9:结束; } |
| 备注 | remark | varchar(64) |  |
| 逻辑删除 | is_delete | unsigned tinyint | { 0:否; 1:是; } |
| 是否可见 | is_visible | unsigned tinyint | { 0:否; 1:是; } |
| 车牌号 | license_plate_number | varchar(16) |  |
| 金额 | amount | decimal(10,2) |  |
| 奖励积分 | rewards_points | unsigned smallint |  | 



### 三、索引
 * 索引命名：小写字母，下划线分割；
 * 长度：不超过 64 个字符；
 * 唯一索引：uk_字段名；
 * 普通索引：idx_字段名；
 * 示例：
   * uk_union_id
   * idx_order_date

