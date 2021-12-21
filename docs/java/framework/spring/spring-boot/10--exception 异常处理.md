> 源码：[https://github.com/langyastudio/langya-tech/tree/springboot/exception](https://github.com/langyastudio/langya-tech/tree/springboot/exception)

- 使用 `@ControllerAdvice` 和 `@ExceptionHandler` 处理全局异常
- 使用 `AbstractErrorController` 处理 **/error** 级别的异常

`@ControllerAdvice` 可以理解为 Controller **共同逻辑的统一处理类**，这样基于 `@ControllerAdvice` 再配合 `@ExceptionHandler`就可以处理全局的异常。



### 定义异常

`EC.java`  枚举类包含异常的错误码与错误信息，方便全局统一使用

```java
public interface IErrorCode
{
    Integer getCode();
    String getMsg();
}


/**
 * 常用API操作码
 */
public enum EC implements IErrorCode{

    /* -----------------成功---------------------------- */
    SUCCESS(111111, "请求成功")
    ,ERROR(111112, "请求异常")

    /* ------------------999999 系统错误--------------------------- */
    , ERROR_SYSTEM_EXCEPTION(999999, "系统异常")

    /* ------------------101xxx 请求错误--------------------------- */
    , ERROR_REQUEST_METHOD_NOT_SUPPORT(101001, "请求方法不支持")
    , ERROR_REQUEST_METHOD_NOT_EXIST(101002, "请求方法不存在")
    , ERROR_REQUEST_API_SIGNATURE_ERROR(101003, "API应用接入签名有误")

    /* ------------------102xxx 参数错误--------------------------- */
    , ERROR_PARAM_NOT_NULL(102001, "参数不能为空")
    , ERROR_PARAM_EXCEPTION(102002, "参数异常")
    , ERROR_PARAM_ILLEGAL(102003, "参数非法")

    /* ------------------103xxx 数据存储错误--------------------------- */
    , ERROR_DATA_ERROR(103000, "数据操作异常")
    , ERROR_DATA_SAVE_FAILURE(103001, "数据保存失败")
    , ERROR_DATA_UPDATE_FAILURE(103002, "数据修改失败")
    , ERROR_DATA_DELETE_FAILURE(103003, "数据删除失败")
    , ERROR_DATA_GET_FAILURE(103004, "数据获取失败")

    /* ------------------104xxx 逻辑层相关的错误--------------------------- */
    //通用
    , ERROR_COMMON_RECORD_NOT_EXIST(104000, "数据不存在")

    /* ------------------106xxx 服务调用相关的错误--------------------------- */
    , ERROR_SERVICE_CALL_EXCEPTION(106001, "服务调用异常")
    , ERROR_SERVICE_RESPONSE_EXCEPTION(106002, "服务响应异常")

    //阿里云
    , ERROR_ALIYUN_SERVICE_SMS_LIMIT(106101, "每分钟只能发送一条短信")

    ;

    /**
     * 返回码
     */
    private final int code;

    /**
     * 提示信息
     */
    private final String msg;

    private EC(int code, String msg)
    {
        this.code = code;
        this.msg  = msg;
    }

    @Override
    public Integer getCode()
    {
        return code;
    }

    @Override
    public String getMsg()
    {
        return msg;
    }
}
```



### 数据返回

`ResultInfo.java` 定义了统一的数据返回类，用于将异常信息、正常信息等统一格式返回给前端

```java
@Data
@Log4j2
public class ResultInfo
{
    private Integer code;
    private String  msg;
    private Long    time;
    private Object  result;

    private ResultInfo(Integer code, String msg, Object result)
    {
        this.setCode(code);
        this.setResult(result);
        this.setMsg(msg);
        this.setTime(System.currentTimeMillis());
    }

    /**
     * 返回正确提示
     *
     * @return Rv
     */
    public static ResultInfo data()
    {
        return data(EC.SUCCESS.getCode(), EC.SUCCESS.getMsg(), null);
    }

    /**
     * 返回正确后的数据
     *
     * @param data 数据
     *
     * @return Rv
     */
    public static ResultInfo data(Object data)
    {
        return data(EC.SUCCESS.getCode(), EC.SUCCESS.getMsg(), data);
    }

    /**
     * 返回状态码和信息
     *
     * @param code 状态码
     * @param msg  信息
     *
     * @return Rv
     */
    public static ResultInfo data(Integer code, String msg)
    {
        return data(code, msg, null);
    }

    /**
     * 返回状态码，信息，数据
     *
     * @param code 状态码
     * @param msg  信息
     * @param data 数据
     *
     * @return Rv
     */
    public static ResultInfo data(Integer code, String msg, Object data)
    {
        return new ResultInfo(code, msg, data);
    }

    /**
     * 返回错误提示
     *
     * @param ec EC
     *
     * @return
     */
    public static ResultInfo data(IErrorCode ec)
    {
        return data(ec.getCode(), ec.getMsg(), null);
    }

    /**
     * 返回错误提示与数据
     *
     * @param ec EC
     *
     * @return
     */
    public static ResultInfo data(IErrorCode ec, String msg)
    {
        return data(ec.getCode(), msg);
    }

    /**
     * 返回错误提示与数据
     *
     * @param ec EC
     *
     * @return
     */
    public static ResultInfo data(IErrorCode ec, Object data)
    {
        return data(ec.getCode(), ec.getMsg(), data);
    }
}

```



### 异常捕获

- 定义 Exception 

  用于处理所有的异常，当未被具体异常捕获时，都会调用该异常的处理逻辑
  
- 定义 MyException

  用于处理自定义的业务异常

- 定义 MethodArgumentNotValidException

  用于处理验失败抛出的异常，这里只是个样例。实际使用中可以定义大量**同类具体**的异常类，用于返回更具体的错误信息，从而提高debug效率

```java
@ControllerAdvice
@Log4j2
public class ExceptionHandle
{
    @Value("${spring.profiles.active}")
    private String env;
    
    // 处理 json 请求体调用接口校验失败抛出的异常
    @ResponseBody
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResultInfo methodArgumentNotValidExceptionHandler(MethodArgumentNotValidException e)
    {
        List<FieldError> fieldErrors = e.getBindingResult().getFieldErrors();
        List<String> collect = new ArrayList<>();
        for(FieldError fieldError :fieldErrors)
        {
            collect.add(fieldError.getField() + fieldError.getDefaultMessage());
        }
        return ResultInfo.data(EC.ERROR_PARAM_EXCEPTION, collect);
    }

    /**
     * 业务异常
     */
    @ResponseBody
    @ExceptionHandler(value = MyException.class)
    public ResultInfo exceptionHandlerMyException(MyException e)
    {
        return this.handlerException(e);
    }

    /**
     * 所有异常
     */
    @ResponseBody
    @ExceptionHandler(Exception.class)
    public ResultInfo handlerException(Exception e)
    {
        String msg  = "";
        Object data = null;
        if ("dev".equals(env))
        {
            msg = e.getMessage();
            msg = msg != null ? msg : e.toString();

            ArrayList<String> ste = new ArrayList<>();
            for (StackTraceElement stackTraceElement : e.getStackTrace())
            {
                String str = stackTraceElement.toString();
                if (!str.contains("hacfin"))
                {
                    continue;
                }

                ste.add(str);
            }
            data = ste;
        }
        else
        {
            msg  = e.getMessage();
            data = null;
        }

        String value = StrUtil.isBlank(msg) ? "系统异常" : msg;

        ResultInfo rtn;
        if(e instanceof MyException)
        {
            rtn = ResultInfo.data(((MyException) e).getCode(), value, data);
        }
        else
        {
            rtn = ResultInfo.data(EC.ERROR_SYSTEM_EXCEPTION.getCode(), value, data);
        }

        logThrowable(e);

        return rtn;
    }   

    /**
     * 打印异常
     */
    public static void logThrowable(Exception e)
    {
        StackTraceElement[] stackTrace = e.getStackTrace();

        log.error(e + "\r\n\t" + e.getMessage());

        StringBuilder stackTraceStr = new StringBuilder();
        for (StackTraceElement stackTraceElement : stackTrace)
        {
            String str = stackTraceElement.toString();
            if (!str.contains("hacfin"))
            {
                continue;
            }

            stackTraceStr.append("\r\n\t").append(str);

        }
        stackTraceStr.append("\r\n");
        log.warn(stackTraceStr.toString());
    }
}
```



**MyException**

继承 `RuntimeException`

```java
public class MyException extends RuntimeException
{
    private Integer code;

    public MyException(IErrorCode ec)
    {
        super(ec.getMsg());
        this.code = ec.getCode();
    }

    public MyException(String msg)
    {
        super(msg);
        this.code = EC.ERROR.getCode();
    }

    public MyException(Integer code, String msg)
    {
        super(msg);
        this.code = code;
    }

    public Integer getCode()
    {
        return code;
    }

    public void setCode(Integer code)
    {
        this.code = code;
    }
}
```



### ErrorController

用于处理 `/error` 异常，例如请求地址不存在

```java
@RestController
public class ErrController extends AbstractErrorController
{
    public static final String ERROR_PATH = "/error";
    
    @Value("${spring.profiles.active}")
    private             String env;

    public ErrController(ErrorAttributes errorAttributes)
    {
        super(errorAttributes);
    }

    @RequestMapping(ERROR_PATH)
    public ResultInfo handleError(HttpServletRequest request)
    {
        WebRequest webRequest = new ServletWebRequest(request);

        String msg;
        Object data = null;

        int       code = EC.ERROR.getCode();
        Throwable e    = (Throwable) webRequest.getAttribute("javax.servlet.error.exception", 0);
        if (e == null)
        {
            Map<String, Object> errorAttributes = getErrorAttributes(request,
                                                                     ErrorAttributeOptions.of(ErrorAttributeOptions.Include.STACK_TRACE));

            Object status = errorAttributes.get("status");
            if(Objects.nonNull(status) && (Integer)status == HttpStatus.HTTP_NOT_FOUND)
            {
                msg = "请求地址不存在";
            }
            else
            {
                msg = "【" + errorAttributes.get("error") + "】" + errorAttributes.get("message");
            }

            if ("dev".equals(env) || "test".equals(env))
            {
                String trace = (String) errorAttributes.get("trace");
                if (trace == null)
                {
                    trace = "";
                }
                data = trace
                        .replace("\r\n", "\n")
                        .replace("\r", "\n")
                        .replace("\t", "")
                        .split("\n");
            }
        }
        else
        {
            if ("dev".equals(env) || "test".equals(env))
            {
                msg = e.getMessage();
                msg = msg != null ? msg : e.toString();

                Integer codeE = (Convert.convert(MyException.class, e)).getCode();
                if (codeE != null)
                {
                    code = codeE;
                }
               
                ArrayList<String> ste = new ArrayList<>();
                for (StackTraceElement stackTraceElement : e.getStackTrace())
                {
                    String str = stackTraceElement.toString();                  
                    ste.add(str);
                }
                data = ste;
            }
            else
            {
                msg = e.getMessage();
                data = null;
            }
        }

        return ResultInfo.data(code, msg, data);
    }
}
```



### 测试

```java
@RestController
@RequestMapping("/api")
public class ApiController
{
    @GetMapping("exception")
    public void exception()
    {
        throw new MyException(EC.ERROR);
    }
}
```

手动抛出 `MyException` 的异常。此时会被 `ExceptionHandle`捕获并返回。

![image-20210906161250769](https://img-note.langyastudio.com/20210906161250.png?x-oss-process=style/watermark)