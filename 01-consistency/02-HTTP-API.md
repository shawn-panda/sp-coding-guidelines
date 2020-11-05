# HTTP API 一致性


### 一、URL

#### 1.1. URL 约定 
 * 风格：小写字母，单词组合使用中杠分割；
 * 语义化：一个功能点对应一个唯一URL，快速识别资源；
 * 支持方法：仅使用 GET/POST，废弃 DELETE/PUT/PATCH；
   * 便于其它功能支持（路由、权限控制等）
 * URL组成：/{version}/{system}/{module}/{operate}
   * 版本：第一位置
   * 系统：第二位置（如系统域名独立，则该级可取消）
   * 模块：第三位置
   * 操作：第四位置
   * PathVar：放置在URL最后；

#### 1.2. URL 示例
 * http://api.company.com/1.0/crm/user/create

#### 1.3. URL 表格

| No. | URL                         | Method | Description      |
|-----|-----------------------------|--------|------------------|
| 01  | /1.0/crm/user/list              | GET    | 查询列表         | 
| 02  | /1.0/crm/user/detail/{userId}   | GET    | 详细信息         |
| 03  | /1.0/crm/user/summary/{userId}  | GET    | 概要信息         |
| 04  | /1.0/crm/user/create            | POST   | 创建             |
| 05  | /1.0/crm/user/modify/{userId}   | POST   | 更新             |
| 06  | /1.0/crm/user/freeze/{userId}   | POST   | 冻结，记日志     | 
| 07  | /1.0/crm/user/unfreeze/{userId} | POST   | 解冻，记日志     | 
| 08  | /1.0/crm/user/delete/{userId}   | POST   | 逻辑删除，记日志 | 



### 二、RESPONSE

#### 2.1. RESPONSE 约定
 * 内容类型：application/json;charset=UTF-8；
 * 字段风格：前后端统一为驼峰法；
 * 统一命名：同一属性字段，所有接口统一命名；
 * 统一格式：
   * 长整型/NULL: 转换为字符串；
   * date: "2020-10-20"；
   * datetime: "2020-10-20T10:00:00Z"；
 * 内容组成：
   * code：状态码，详见 [[错误码设计](../02-error-code/README.md)]
   * message：错误描述，详见 [[错误码设计](../02-error-code/README.md)]
   * data：返回数据
   * enum：字典/枚举值

#### 2.2. Response 示例
```json
{
    "code": "000000",
    "message": "用户信息创建成功",
    "data": {
        "userId": "120000000001",
        "userName": "陈小春",
        "userPhone": "18588221888",
        "userGender": "MALE",
        "userStatus": "1",
        "userBirthday": "1998-06-04",
        "gmtCreate": "2020-10-20T10:12:23Z",
        "gmtModified": "2020-10-20T10:12:23Z"
    },
    "enum": {
        "eUserGender": {
            "MALE": "男",
            "FEMALE": "女",
            "UNKNOWN": "未知"
        },
        "eUserStatus": {
            "0": "未激活",
            "1": "正常",
            "2": "冻结",
            "9": "注销"
        }
    }
}
``` 


