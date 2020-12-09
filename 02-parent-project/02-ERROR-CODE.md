# 错误码


### 一、错误码说明

#### 1.1. 格式说明
  * 6位字符串
  * 统一格式：MM000X
  * MM : 项目模块
  * 000X : 错误编号，自增

#### 1.2. 成功
 * 000000 : SUCCESS

#### 1.3. MM: 项目模块
 * 11 : 公共/common;
 * 12 : 用户/user;
 * 13 : 消息/message;
 * 14 : 券类/coupon;
 * 15 : 积分/point;


### 二、代码组织
#### 2.1. IErrorCode.java

```java
public interface IErrorCode {
    public String getCode();
    public String getMessage();
}
```

#### 2.2. IErrorCodeCommon.java

```java
public interface IErrorCodeCommon extends IErrorCode {
    String M_COMMON = "11";
}
```

#### 2.3. EErrorCodeCommon.java

```java
public enum EErrorCodeCommon implements IErrorCodeCommon {
    ERR_SUCCESS("000000", "SUCCESS"),
    ERR_PARAM_EMPTY(M_COMMON + "0001", "请求参数为空"),
    ERR_PARAM_INVALID(M_COMMON + "0002", "请求参数无效"),
    ERR_REMOTE_API_FAILED(M_COMMON + "0003", "远程接口调用失败"),
    ERR_DELETE_FAILED(M_COMMON + "0004", "删除失败");

    //...
```

#### 2.4. BizServiceException.java 
 > 业务服务异常处理

```java
public class BizServiceException extends RuntimeException {
    private IErrorCode iErrorCode;
    private String errorCode;
    private String errorMessage;

    public BizServiceException(IErrorCode iErrorCode) {
        super();
        this.iErrorCode = iErrorCode;
        this.errorCode = iErrorCode.getCode();
        this.errorMessage = iErrorCode.getMessage();
    }

    //...
```


### 三、使用示例

#### 3.1. 打印输出

```java
System.out.println(EErrorCodeCommon.ERR_PARAM_EMPTY.getCode());
```

#### 3.2. 抛出异常，BizServiceException.java，

```java
throw new BizServiceException(EErrorCodeCommon.ERR_REMOTE_API_FAILED);
```