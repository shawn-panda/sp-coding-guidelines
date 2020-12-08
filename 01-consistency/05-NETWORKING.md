# 环境与网络


### 一、环境

#### 1.1. 主环境

| slug  | 环境     | 网络 | 数据库   | 服务组件 |
|-------|----------|------|----------|----------|
| DEV   | 开发环境 | 独立 | 独立     | 独立     |
| UAT   | 测试环境 | 独立 | 独立     | 独立     |
| PRD   | 生产环境 | 独立 | 独立     | 独立     |


#### 1.2. 副环境

| slug  | 环境     | 网络 | 数据库   | 服务组件 |
|-------|----------|------|----------|----------|
| LOCAL | 本地开发 | 独立 | 共享开发 | 共享开发 |
| BETA  | 灰度环境 | 独立 | 共享生产 | 共享生产 |


#### 1.3. 建议
 * 尽量减少环境，减少配置、维护成本，保障环境稳定；
 * 数据/服务共享环境，如生产和灰度，考虑对接第三方/回调的一致；



### 二、网络
 * 除生产无标识外，其余环境需要标识；
 * 除生产API、CDN外，其余资源均只允许内网/堡垒机访问；

| 资源 \ 环境 | DEV                       | UAT                       | BETA                  | PRD                   |
|-------------|---------------------------|---------------------------|-----------------------|-----------------------|
| API         | api-dev.demo.com          | api-uat.demo.com          | api-beta.demo.com     | api.demo.com          |
| MySQL       | mydb-dev.demo.com:3306    | mydb-uat.demo.com:3306    | mydb.demo.com:3306    | mydb.demo.com:3306    |
| MongoDB     | mgdb-dev.demo.com:3306    | mgdb-uat.demo.com:3306    | mgdb.demo.com:3306    | mgdb.demo.com:3306    |
| Redis       | redis-dev.demo.com:6379   | redis-uat.demo.com:6379   | redis.demo.com:6379   | redis.demo.com:6379   |
| CDN         | cdn.demo.com              | cdn.demo.com              | cdn.demo.com          | cdn.demo.com          |
