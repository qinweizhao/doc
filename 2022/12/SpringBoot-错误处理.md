# SpringBoot-错误处理

默认情况下，SpringBoot 提供`/error`处理所有错误的映射

## 一、默认规则

- 机器客户端，它将生成 JSON 响应，其中包含错误，HTTP 状态和异常消息的详细信息。

  ![2022-12-30_141620](https://img.qinweizhao.com/2022/12/2022-12-30_141620.png)

- 浏览器客户端，响应一个 “whitelabel” 错误视图，以 HTML 格式呈现相同的数据。

  ![2022-12-30_142015](https://img.qinweizhao.com/2022/12/2022-12-30_142015.png)

## 二、相关组件

核心配置类：**ErrorMvcAutoConfiguration**

### 1、DefaultErrorAttributes

**定义错误页面中可以包含哪些数据**

![2022-12-30_144442](https://img.qinweizhao.com/2022/12/2022-12-30_144442.png)

### 2、BasicErrorController

**管理错误相应逻辑，返回 JSON 或者错误视图**

![2022-12-30_144903](https://img.qinweizhao.com/2022/12/2022-12-30_144903.png)

### 3、DefaultErrorViewResolver

**如果发生错误，会以 HTTP 的状态码作为视图页地址，找到真正的页面**

![2022-12-30_150456](https://img.qinweizhao.com/2022/12/2022-12-30_150456.png)

在导入模版引擎的 starter 情况下，templates 下面 error 文件夹中的 4xx，5xx 页面会被自动解析。

## 三、处理流程

测试代码：

```java
    @RequestMapping("test-error")
    public void testError() {
        int a = 1 / 0;
        System.out.println(a);
    }
```

 1、目标方法运行期间有任何异常都会被 catch、并将异常赋给 dispatchException 。

![2022-12-30_151836](https://img.qinweizhao.com/2022/12/2022-12-30_151836.png)

2、调用 `processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);` 处理结果。

![2022-12-30_152052](https://img.qinweizhao.com/2022/12/2022-12-30_152052.png)

3、调用 `processHandlerException()` 方法，遍历系统中所有的异常处理解析器，哪个解析器返回结果不为 null，就结束循环。

![2022-12-30_152222](https://img.qinweizhao.com/2022/12/2022-12-30_152222.png)

![2022-12-30_152358](https://img.qinweizhao.com/2022/12/2022-12-30_152358.png)

4、遍历完所有解析器，我们发现他们都不能返回一个不为空的 `ModelAndView` 对象，于是它会继续抛出异常。

![2022-12-30_152623](https://img.qinweizhao.com/2022/12/2022-12-30_152623.png)

5、当系统发现没有任何人能处理这个异常时，底层就会发送 /error 请求。然后被 BasicErrorController 处理。

![2022-12-30_152745](https://img.qinweizhao.com/2022/12/2022-12-30_152745.png)

## 四、补充

1、@ControllerAdvice+@ExceptionHandler 处理全局异常；底层是 **ExceptionHandlerExceptionResolver 支持的**。

2、@ResponseStatus+自定义异常 ；底层是 **ResponseStatusExceptionResolver 支持的**。 

3、Spring 底层的异常，如参数类型转换异常；底层是 **DefaultHandlerExceptionResolver 支持的**。 

4、当异常无法处理。tomcat 底层就会 response.sendError。error 请求就会转给 controller。basicErrorController 要去的页面地址是 **ErrorViewResolver** 。


## 附

>代码地址：
>
>https://github.com/qinweizhao/qwz-sample/tree/master/framework/fsb-error
