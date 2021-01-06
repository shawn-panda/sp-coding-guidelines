# 公共组件


### 一、对象存储（CDN / OSS）
 * 须有目录规划: 标识文件所属功能，以及限制单个文件夹文件数量；
 * 须有公私分区: 权限隔离；
 * 前端直接上传OSS，返回URL给后端； 
 * UI资源文件规划: 
   * 规范: /public/asserts/{version}/{system}/[css|js|img]/{unique-name}.png
   * 示例: /public/asserts/1.0/crm/img/logo-128px.png
 * 功能型文件规划:
   * 规范: /private/{system}/{module}/{sub-module}/{date}/{hash-name}.jpg
   * 示例: /private/crm/order/comments/20201212/30a16bf311b172be6180d5e6375dbc90.jpg



### 二、缓存（Redis）

#### 2.1. 主键规划
 * 命名：统一大写字母，英文冒号“:”分割；
 * 长度：不超过 64 个字符；
 * 须按有效期规划: 三位标识，头位为时间单位；{ M:分钟; H:小时; D:天; LCK:锁; }
   * M05: 5分钟有效
   * M60: 1小时有效
   * H24: 24小时有效
   * D15: 15天有效
   * LCK: 锁
 * 须按系统、模块规划；
 * 示例: 
   * M60:CRM:USER:ID:MAP
   * LCK:CRM:POINT:PAYMENT:${orderNo}

#### 2.2. 数据类型
 * 略



### 三、消息队列（MQ）
 * MQ两大使用场景: 应用解耦，以及流量削锋；
 * 尽量少使用MQ；（控制成本，MQ很贵，会增加系统复杂度）
 * 减少创建Topic，尽量采用Tag；（控制成本，Topic很贵）



### 四、短信、邮件
 * 略



### 五、开放平台
 * 维护第三方，扩展自身平台能力
 * 减少自身开发工作



