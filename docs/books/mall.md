# 开源项目mall 源码阅读笔记
[mall-common](#mall-common)

### mall-common 
该模块存放其他模块都可能调用的类 主要是对`api`结果的封装，`exception` 异常的封装，统一管理   

#### `exception`模块
定义了Assert类，其它类调用该类抛出异常
```

public class Asserts {
    public static void fail(String message) {
        throw new ApiException(message);
    }

    public static void fail(IErrorCode errorCode) {
        throw new ApiException(errorCode);
    }
}
```
再定义了`GlobalExceptionHandler`类，其它类调用该类抛出异常 该类专门捕获`ApiException`异常，并对其进行处理
`@ResponseBody` 这个注解**一定要添加** 这个注解是会将return的结果返回给`api`的调用方
```
@ControllerAdvice
public class GlobalExceptionHandler {

    @ResponseBody   // 这个注解必须要 没有它程序不会返回给用户
    @ExceptionHandler(value = ApiException.class)
    public CommonResult handle(ApiException e) {
        if (e.getErrorCode() != null) {
            return CommonResult.failed(e.getErrorCode());
        }
        return CommonResult.failed(e.getMessage());
    }
}
```
`ApiException`异常类的定义
```
public class ApiException extends RuntimeException {
    private IErrorCode errorCode;

    public ApiException(IErrorCode errorCode) {
        super(errorCode.getMessage());
        this.errorCode = errorCode;
    }
```
在service层调用断言`Asserts.fail`
```
        if (!hasStock(cartPromotionItemList)) {
            Asserts.fail("库存不足，无法下单");
        }
```
小知识点 在所有涉及到订单类的处理逻辑方法，都设计成`事务`的方式  且在**接口**进行声明
```
public interface OmsPortalOrderService {
    /**
     * 根据用户购物车信息生成确认单信息
     */
    ConfirmOrderResult generateConfirmOrder();

    /**
     * 根据提交信息生成订单
     */
    @Transactional
    Map<String, Object> generateOrder(OrderParam orderParam);
```
#### `api`模块
api接口模块是对**所有返回结果**进行的封装，增加`status`和`message`字段(状态解释)  提供给用户更多有用信息

在设计该类的时候，由于这是通用的类，并不知道传入进来的data的类型，所以该类是`模板类`
```
public class CommonResult<T> {
    private long code;
    private String message;
    private T data;

    protected CommonResult() {
    }

    protected CommonResult(long code, String message, T data) {
        this.code = code;
        this.message = message;
        this.data = data;
    }

    /**
     * 成功返回结果
     *
     * @param data 获取的数据
     */
    public static <T> CommonResult<T> success(T data) {
        return new CommonResult<T>(ResultCode.SUCCESS.getCode(), ResultCode.SUCCESS.getMessage(), data);
    }
    
```




