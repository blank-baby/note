#### 1.HandlerMapping

请求会先查找HandlerMapping，在找到之后返回的是HandlerExecutionChain

注意点：

- HandlerExecutionChain包含
  - HandlerMapping
  - 过滤器(HandlerInterceptor的过滤器会被包含在MappedInterceptor这个类里面)





2.HandlerAdapter

主要处理HandlerMapping的方法方法调用。例如从方法的参数从请求体中获取参数

HandlerAdapter

```java
/**
 *  请求的参数和方法中参数对应
 *  执行controller中的方法
 */
invocableMethod.invokeAndHandle(webRequest, mavContainer);

/**
 *  获取ModelAndView
 */
return getModelAndView(mavContainer, modelFactory, webRequest);
		
```

HandlerMethod

```java
/**
 * 直接调用controller的方法
 */
Object returnValue = invokeForRequest(webRequest, mavContainer, providedArgs);

/**
 * 处理方法的返回值
 * 如果方法带responcebody  直接返回
 */
this.returnValueHandlers.handleReturnValue(
					returnValue, getReturnValueType(returnValue), mavContainer, webRequest);
```

invokeForRequest()

```java
/**
 * 获取controller方法所需要的参数
 */
Object[] args = getMethodArgumentValues(request, mavContainer, providedArgs);

/**
 * 执行controller的方法
 */
return doInvoke(args);
```

getMethodArgumentValues()

```java
MethodParameter[] parameters = getMethodParameters();
if (ObjectUtils.isEmpty(parameters)) {
    return EMPTY_ARGS;
}

Object[] args = new Object[parameters.length];
for (int i = 0; i < parameters.length; i++) {
    MethodParameter parameter = parameters[i];
    parameter.initParameterNameDiscovery(this.parameterNameDiscoverer);
    args[i] = findProvidedArgument(parameter, providedArgs);
    if (args[i] != null) {
        continue;
    }
    if (!this.resolvers.supportsParameter(parameter)) {
        throw new IllegalStateException(formatArgumentError(parameter, "No suitable resolver"));
    }
    try {
        /**
		*  从请求体中寻找参数并且把它转换成需要的类型
		*/
        args[i] = this.resolvers.resolveArgument(parameter, mavContainer, request, this.dataBinderFactory);
    }
    catch (Exception ex) {
        // Leave stack trace for later, exception may actually be resolved and handled...
        if (logger.isDebugEnabled()) {
            String exMsg = ex.getMessage();
            if (exMsg != null && !exMsg.contains(parameter.getExecutable().toGenericString())) {
                logger.debug(formatArgumentError(parameter, exMsg));
            }
        }
        throw ex;
    }
}
		return args;
```

resolveArgument()

```java

```





