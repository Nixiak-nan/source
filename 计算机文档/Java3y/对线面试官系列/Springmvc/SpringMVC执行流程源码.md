## 前言

> 只有光头才能变强。

> **文本已收录至我的GitHub精选文章，欢迎Star**：[https://github.com/ZhongFuCheng3y/3y](https://github.com/ZhongFuCheng3y/3y)



这篇`SpringMVC`被催了很久了，这阵子由于做整合系统的事，所以非常非常地忙。这周末早早就回了公司肝这篇文章了。



如果关注三歪的同学会发现，三歪最近写的很多文章都是结合了现有的系统去写的。这些问题都是真实开发场景会遇到的、用的上的，这些案例对未工作的同学帮助应该还是蛮大的。



不多BB了，还是进入今天的正题吧「**SpringMVC**」

## 先简单聊聊SpringMVC

如果你们玩知乎，很可能会看到我的身影。我经常会去知乎水回答。在知乎有很多初学者都会问的一个问题：「**我学习SpringMVC需要什么样的基础**」



我一定会让他们先学Servlet，再学SpringMVC的。虽然说我们在现实开发中几乎不会写原生Servlet的代码了，但我始终认为学完Servlet再学SpringMVC，对理解SpringMVC是有好处的。



> 三歪题外话：我当时在学SpringMVC之前其实已经接触过另外一个web框架（当然了Servlet也是学了的），那就是「大名鼎鼎」的Struts2。只要是Struts2有的功能，SpringMVC都会有。
>
> 当时初学Struts2的时候用的是XML配置的方式去开发的，再转到SpringMVC注解的时候，觉得SpringMVC真香。
>
> Struts2在2020年已经不用学了，**学SpringMVC的基础是Servlet，只要Servlet基础还行，上手SpringMVC应该不成问题。**



从Servlet到SpringMVC，你会发现SpringMVC帮我们做了很多的东西，我们的代码肯定是没以前多了。



**Servlet：**

我们以前可能需要将传递进来的参数**手动**封装成一个Bean，然后继续往下传：

![](https://tva1.sinaimg.cn/large/007S8ZIlgy1geu53hnlk2j30w80ie42l.jpg)

**SpringMVC:**

现在SpringMVC**自动**帮我们将参数封装成一个Bean

![](https://tva1.sinaimg.cn/large/007S8ZIlgy1gevhejtpw8j30si0gejv3.jpg)

**Servlet:**

以前我们要导入其他的`jar`包去手动处理**文件上传**的细节：

![](https://tva1.sinaimg.cn/large/007S8ZIlgy1geu5najavij30u00vynbd.jpg)

**SpringMVC：**

现在SpringMVC上传文件用一个MultipartFile对象都给我们封装好了

![](https://tva1.sinaimg.cn/large/007S8ZIlgy1geu83oneugj31560eejun.jpg)



........



说白了，在Servlet时期我们这些活都能干，只不过SpringMVC把很多东西都给屏蔽了，于是我们用起来就更加舒心了。



在学习SpringMVC的时候实际上也是学习这些功能是怎么用的而已，并不会太难。这次整理的SpringMVC电子书其实也是在讲SpringMVC是如何使用的

- 比如说传递一个日期字符串来，SpringMVC默认是不能转成日期的，那我们可以怎么做来实现。
- SpringMVC的文件上传是怎么使用的
- SpringMVC的拦截器是怎么使用的
- SpringMVC是怎么对参数绑定的
- ......

![](https://tva1.sinaimg.cn/large/007S8ZIlgy1geucf9xu7zj31h20u0ki5.jpg)



现在「电子书」已经放出来了，但是**别急**，重头戏在后面。显然，通过上面的电子书是可以知道SpringMVC**是怎么用的**。



但是这在面试的时候人家是不会问你SpringMVC的一些用法的，而SpringMVC面试问得最多的就是：**SpringMVC请求处理的流程是怎么样的**。



其实也很简单，流程就是下面这张图：

![](https://tva1.sinaimg.cn/large/007S8ZIlgy1geucv72pffj31dq0t014f.jpg)



再简化一点，可以发现流程不复杂

![](https://tva1.sinaimg.cn/large/007S8ZIlgy1geud790kiyj30pp09x3zc.jpg)

在面试的时候甚至能一句话就讲完了，但这够吗，这是面试官想要的吗？那肯定不是。那我们想知道SpringMVC是做了什么吗？想的吧（**不管你们想不想，反正三歪想看**）。

![](https://tva1.sinaimg.cn/large/007S8ZIlgy1gevb1edk91j30je0l8jud.jpg)



**由于想要主流程更加清晰一点，我会在源码添加部分注释以及删减部分的代码**



> 以@ResponseBody和@RequestBody的Controller代码讲解为主，这是线上环境用得最多的

## DispatcherServlet源码

首先我们看看DispatcherServlet的类结构，可以清楚地发现实际DispatcherServlet就是Servlet接口的一个子类（这也就是为什么网上这么多人说DispatcherServlet的原理实际上就是Servlet）

![](https://tva1.sinaimg.cn/large/007S8ZIlgy1geudm9v57wj31jk0p20uu.jpg)



我们在DispatcherServlet类上可以看到很多熟悉的成员变量（**组件**），所以看下来，我们要的东西，DispatcherServlet可**全都有**。

```java
// 文件处理器
private MultipartResolver multipartResolver;

// 映射器
private List<HandlerMapping> handlerMappings;

// 适配器
private List<HandlerAdapter> handlerAdapters;

// 异常处理器
private List<HandlerExceptionResolver> handlerExceptionResolvers;

// 视图解析器
private List<ViewResolver> viewResolvers;
```

然后我们会发现它们在`initStrategies()`上初始化：

```java
protected void initStrategies(ApplicationContext context) {
  initMultipartResolver(context);
  initLocaleResolver(context);
  initThemeResolver(context);
  initHandlerMappings(context);
  initHandlerAdapters(context);
  initHandlerExceptionResolvers(context);
  initRequestToViewNameTranslator(context);
  initViewResolvers(context);
  initFlashMapManager(context);
}
```

请求进到DispatcherServlet，其实全部都会打到`doService()`方法上。我们看看这个`doService()`方法做了啥：

```java
protected void doService(HttpServletRequest request, HttpServletResponse response) throws Exception {
		
		// 设置一些上下文...(省略一大部分)
		request.setAttribute(OUTPUT_FLASH_MAP_ATTRIBUTE, new FlashMap());
		request.setAttribute(FLASH_MAP_MANAGER_ATTRIBUTE, this.flashMapManager);

		try {
      // 调用doDispatch
			doDispatch(request, response);
		}
		finally {
			if (!WebAsyncUtils.getAsyncManager(request).isConcurrentHandlingStarted()) {
				if (attributesSnapshot != null) {
					restoreAttributesAfterInclude(request, attributesSnapshot);
				}
			}
		}
	}
```

所以请求会走到`doDispatch(request, response);`里边，我们再进去看看：

```java
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
   HttpServletRequest processedRequest = request;
   HandlerExecutionChain mappedHandler = null;
   boolean multipartRequestParsed = false;
   WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);

   try {
      ModelAndView mv = null;
      Exception dispatchException = null;

      try {
         // 检查是不是文件上传请求
         processedRequest = checkMultipart(request);
         multipartRequestParsed = (processedRequest != request);

         // 找到HandlerExecutionChain
         mappedHandler = getHandler(processedRequest);
         if (mappedHandler == null || mappedHandler.getHandler() == null) {
            noHandlerFound(processedRequest, response);
            return;
         }
         // 得到对应的hanlder适配器
         HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());

         // 拦截前置处理
         if (!mappedHandler.applyPreHandle(processedRequest, response)) {
            return;
         }

         // 真实处理请求
         mv = ha.handle(processedRequest, response, mappedHandler.getHandler());

         // 视图解析器处理
         applyDefaultViewName(processedRequest, mv);
        
         // 拦截后置处理
         mappedHandler.applyPostHandle(processedRequest, response, mv);
      }
      catch (Exception ex) {
         dispatchException = ex;
      }
   }
}
```

这里的流程跟我们上面的图的流程几乎是一致的了。我们从源码可以知道的是，原来SpringMVC的拦截器是在MappingHandler的时候一齐返回的，返回的是一个`HandlerExecutionChain`对象。这个对象也不难，我们看看：

```java
public class HandlerExecutionChain {

	private static final Log logger = LogFactory.getLog(HandlerExecutionChain.class);

  // 真实的handler
	private final Object handler;

  // 拦截器List
	private HandlerInterceptor[] interceptors;
	private List<HandlerInterceptor> interceptorList;

	private int interceptorIndex = -1;
}
```

OK，整体的流程我们是已经看完了，顺便要不我们去看看它是怎么找到handler的？**三歪带着你们冲！**我们点进去`getHandler()`后，发现它就把默认实现的Handler遍历一遍，然后选出合适的：

```java
protected HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception {
	// 遍历一遍默认的Handler实例，选出合适的就返回
  for (HandlerMapping hm : this.handlerMappings) {
    HandlerExecutionChain handler = hm.getHandler(request);
    if (handler != null) {
      return handler;
    }
  }
  return null;
}
```

再进去`getHandler`里边看看呗，里边又有几层，我们最后可以看到它根据**路径**去匹配，走到了`lookupHandlerMethod`这么一个方法

```java
protected HandlerMethod lookupHandlerMethod(String lookupPath, HttpServletRequest request) throws Exception {
		List<Match> matches = new ArrayList<Match>();
  	// 获取路径
		List<T> directPathMatches = this.mappingRegistry.getMappingsByUrl(lookupPath);

  	// 对匹配的排序，找到最佳匹配的
		if (!matches.isEmpty()) {
			Comparator<Match> comparator = new MatchComparator(getMappingComparator(request));
			Collections.sort(matches, comparator);
			if (logger.isTraceEnabled()) {
				logger.trace("Found " + matches.size() + " matching mapping(s) for [" +
						lookupPath + "] : " + matches);
			}
			Match bestMatch = matches.get(0);
			if (matches.size() > 1) {
				if (CorsUtils.isPreFlightRequest(request)) {
					return PREFLIGHT_AMBIGUOUS_MATCH;
				}
				Match secondBestMatch = matches.get(1);
				if (comparator.compare(bestMatch, secondBestMatch) == 0) {
					Method m1 = bestMatch.handlerMethod.getMethod();
					Method m2 = secondBestMatch.handlerMethod.getMethod();
					throw new IllegalStateException("Ambiguous handler methods mapped for HTTP path '" +
							request.getRequestURL() + "': {" + m1 + ", " + m2 + "}");
				}
			}
			handleMatch(bestMatch.mapping, lookupPath, request);
			return bestMatch.handlerMethod;
		}
		else {
			return handleNoMatch(this.mappingRegistry.getMappings().keySet(), lookupPath, request);
		}
	}
```

找拦截器大概也是上面的一个过程，于是我们就可以顺利拿到`HandlerExecutionChain`了，找到`HandlerExecutionChain`后，我们是先去拿对应的`HandlerAdaptor`。我们也去看看里边做了什么：

```java
// 遍历HandlerAdapter实例，找到个合适的返回
protected HandlerAdapter getHandlerAdapter(Object handler) throws ServletException {
		for (HandlerAdapter ha : this.handlerAdapters) {
			if (ha.supports(handler)) {
				return ha;
			}
		}
	}
```

我们看一个常用HandlerAdapter实例`RequestMappingHandlerAdapter`，会发现他会初始化很多的参数解析器，其实我们经常用的`@ResponseBody`解析器就被内置在里边：

```java
private List<HandlerMethodArgumentResolver> getDefaultArgumentResolvers() {
		List<HandlerMethodArgumentResolver> resolvers = new ArrayList<HandlerMethodArgumentResolver>();

		resolvers.add(new MatrixVariableMethodArgumentResolver());
		resolvers.add(new MatrixVariableMapMethodArgumentResolver());
		resolvers.add(new ServletModelAttributeMethodProcessor(false));
  	// ResponseBody Requestbody解析器
		resolvers.add(new RequestResponseBodyMethodProcessor(getMessageConverters(), this.requestResponseBodyAdvice));
		resolvers.add(new RequestPartMethodArgumentResolver(getMessageConverters(), t
		// 等等
		return resolvers;
	}
```

得到HandlerAdaptor后，随之而行的就是拦截器的前置处理，然后就是真实的`mv = ha.handle(processedRequest, response, mappedHandler.getHandler())`。

这里边嵌套了好几层，我就不一一贴代码了，我们会进入`ServletInvocableHandlerMethod#invokeAndHandle`方法，我们看一下这里边做了什么：

```java
public void invokeAndHandle(ServletWebRequest webRequest,
			ModelAndViewContainer mavContainer, Object... providedArgs) throws Exception {

  	// 处理请求
		Object returnValue = invokeForRequest(webRequest, mavContainer, providedArgs);
		setResponseStatus(webRequest);

		if (returnValue == null) {
			if (isRequestNotModified(webRequest) || hasResponseStatus() || mavContainer.isRequestHandled()) {
				mavContainer.setRequestHandled(true);
				return;
			}
		}
  	//.. 

		mavContainer.setRequestHandled(false);
		try {
      // 处理返回值
			this.returnValueHandlers.handleReturnValue(
					returnValue, getReturnValueType(returnValue), mavContainer, webRequest);
		}
	}
```

处理请求的方法我们进去看看`invokeForRequest`

```java
public Object invokeForRequest(NativeWebRequest request, ModelAndViewContainer mavContainer,
			Object... providedArgs) throws Exception {
		
  	// 得到参数
		Object[] args = getMethodArgumentValues(request, mavContainer, providedArgs);
		
  	// 调用方法
		Object returnValue = doInvoke(args);
		if (logger.isTraceEnabled()) {
			logger.trace("Method [" + getMethod().getName() + "] returned [" + returnValue + "]");
		}
		return returnValue;
	}
```

我们看看它是怎么处理参数的，`getMethodArgumentValues`方法进去看看：

```java
private Object[] getMethodArgumentValues(NativeWebRequest request, ModelAndViewContainer mavContainer,
			Object... providedArgs) throws Exception {

  	// 得到参数
		MethodParameter[] parameters = getMethodParameters();
		Object[] args = new Object[parameters.length];
		for (int i = 0; i < parameters.length; i++) {
			MethodParameter parameter = parameters[i];
			parameter.initParameterNameDiscovery(this.parameterNameDiscoverer);
			GenericTypeResolver.resolveParameterType(parameter, getBean().getClass());
			args[i] = resolveProvidedArgument(parameter, providedArgs);
			if (args[i] != null) {
				continue;
			}
      // 找到适配的参数解析器
			if (this.argumentResolvers.supportsParameter(parameter)) {
				try {
					args[i] = this.argumentResolvers.resolveArgument(
							parameter, mavContainer, request, this.dataBinderFactory);
					continue;
				}
				//.....
		}
		return args;
	}
```

这些参数解析器实际上在HandlerAdaptor内置的那些，这里不好放代码，所以我截个图吧：

![](https://tva1.sinaimg.cn/large/007S8ZIlgy1gevfth1ajuj31uq0u0at3.jpg)

针对于RequestResponseBodyMethodProcessor解析器我们看看里边做了什么：

```java
public Object resolveArgument(MethodParameter parameter, ModelAndViewContainer mavContainer,
			NativeWebRequest webRequest, WebDataBinderFactory binderFactory) throws Exception {

    // 通过Converters对参数转换
		Object arg = readWithMessageConverters(webRequest, parameter, parameter.getGenericParameterType());
		String name = Conventions.getVariableNameForParameter(parameter);

		WebDataBinder binder = binderFactory.createBinder(webRequest, arg, name);
		// ...
		mavContainer.addAttribute(BindingResult.MODEL_KEY_PREFIX + name, binder.getBindingResult());

		return arg;
	}
```

再进去`readWithMessageConverters`里边看看：

```java
protected <T> Object readWithMessageConverters(HttpInputMessage inputMessage, MethodParameter param,
			Type targetType) throws IOException, HttpMediaTypeNotSupportedException, HttpMessageNotReadableException {

		// ...处理请求头

		try {
			inputMessage = new EmptyBodyCheckingHttpInputMessage(inputMessage);

      // HttpMessageConverter实例去对参数转换
			for (HttpMessageConverter<?> converter : this.messageConverters) {
				Class<HttpMessageConverter<?>> converterType = (Class<HttpMessageConverter<?>>) converter.getClass();
				if (converter instanceof GenericHttpMessageConverter) {
					GenericHttpMessageConverter<?> genericConverter = (GenericHttpMessageConverter<?>) converter;
					if (genericConverter.canRead(targetType, contextClass, contentType)) {
						if (logger.isDebugEnabled()) {
							logger.debug("Read [" + targetType + "] as \"" + contentType + "\" with [" + converter + "]");
						}
						if (inputMessage.getBody() != null) {
							inputMessage = getAdvice().beforeBodyRead(inputMessage, param, targetType, converterType);
							body = genericConverter.read(targetType, contextClass, inputMessage);
							body = getAdvice().afterBodyRead(body, inputMessage, param, targetType, converterType);
						}
						else {
							body = null;
							body = getAdvice().handleEmptyBody(body, inputMessage, param, targetType, converterType);
						}
						break;
					}
				}
				//...各种判断
		

		return body;
	}
```

看到这里，有没有看不懂，想要退出的感觉了？？别慌，三歪带你们看看这份熟悉的配置：

```xml
<!-- 启动JSON返回格式 -->
	<bean	class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter">
		<property name="messageConverters">
			<list>
				<ref bean="jacksonMessageConverter" />
			</list>
		</property>
	</bean>
	<bean id="jacksonMessageConverter"
		class="org.springframework.http.converter.json.MappingJackson2HttpMessageConverter">
		<property name="supportedMediaTypes">
			<list>
				<value>text/html;charset=UTF-8</value>
				<value>application/json;charset=UTF-8</value>
				<value>application/x-www-form-urlencoded;charset=UTF-8</value>
			</list>
		</property>
		<property name="objectMapper" ref="jacksonObjectMapper" />
	</bean>
	<bean id="jacksonObjectMapper" class="com.fasterxml.jackson.databind.ObjectMapper" />

```

我们在SpringMVC想要使用`@ResponseBody`返回JSON格式都会在配置文件上配置上面的配置，`RequestMappingHandlerAdapter`这个适配器就是上面所说的那个，内置了`RequestResponseBodyMethodProcessor`解析器，然后`MappingJackson2HttpMessageConverter`实际上就是`HttpMessageConverter`接口的实例

![](https://tva1.sinaimg.cn/large/007S8ZIlgy1gevg9t9oywj317m0h8abj.jpg)

然后在返回的时候也经过HttpMessageConverter去将参数转换后，写给HTTP响应报文。转换的流程大致如图所示：

![img](https://tva1.sinaimg.cn/large/007S8ZIlgy1gevgd6l3i6j30ro0e9gnd.jpg)

视图解析器后面就不贴了，大概的流程就如上面的源码，我再画个图来加深一下理解吧：

![](https://tva1.sinaimg.cn/large/007S8ZIlgy1gevgx3tcktj30uu0m5whw.jpg)

## 最后

SpringMVC我们使用的时候非常简便，在内部实际上帮我们做了很多(有各种的HandlerAdaptor)，SpringMVC的请求流程面试的时候还是面得很多的，还是可以看看源码它帮我们做了什么，过一遍可能会发现自己能看懂以前的配置了。



现在已经工作有一段时间了，为什么还来写`SpringMVC`呢，原因有以下几个：

- 我是一个对**排版**有追求的人，如果早期关注我的同学可能会发现，我的GitHub、文章导航的`read.me`会经常更换。现在的[GitHub](https://github.com/ZhongFuCheng3y/3y)导航也不合我心意了（太长了），并且早期的文章，说实话排版也不太行，我决定重新搞一波。
- 我的文章会分发好几个平台，但文章发完了可能就没人看了，并且图床很可能因为平台的防盗链就挂掉了。又因为有很多的读者问我：”**你能不能把你的文章转成PDF啊**？“
- 我写过很多系列级的文章，这些文章就几乎不会有太大的改动了，就非常适合把它们给”**持久化**“。



基于上面的原因，我决定把我的系列文章汇总成一个`PDF/HTML/WORD/epub`文档。说实话，打造这么一个文档**花了我不少的时间**。为了防止**白嫖**，关注我的公众号回复「**888**」即可获取。



**SpringMVC电子书，有兴趣的同学可以浏览一波。共有「47」页**

![](https://tva1.sinaimg.cn/large/007S8ZIlgy1geucf9xu7zj31h20u0ki5.jpg)





参考资料：

- [https://www.cnblogs.com/java-chen-hao/category/1503579.html](https://www.cnblogs.com/java-chen-hao/category/1503579.html)
- [https://www.jianshu.com/p/1bff57c74037](https://www.jianshu.com/p/1bff57c74037)
- [https://stackoverflow.com/questions/18682486/why-does-spring-mvc-need-at-least-two-contexts](https://stackoverflow.com/questions/18682486/why-does-spring-mvc-need-at-least-two-contexts)

### 各类知识点总结

> 下面的文章都有对应的**原创精美**PDF，在持续更新中，可以来找我催更~

- [92页的Mybatis](https://mp.weixin.qq.com/s/0_zTBooRV4RTWQa8VOwiWg)
- [129页的多线程](https://mp.weixin.qq.com/s/r7IrmvBxG5W0hswfcgFjcQ)
- [141页的Servlet](https://mp.weixin.qq.com/s?__biz=MzI4Njg5MDA5NA==&mid=2247486798&idx=1&sn=ce900e97a495ffd681cd0ad9b78aa5ca&chksm=ebd74c4fdca0c559d0a32a3f3ddb3f579d3a16b47f70234c46ac2e5df315e7df90f93d1715b9&token=1109491988&lang=zh_CN#rd)
- [158页的JSP](https://mp.weixin.qq.com/s?__biz=MzI4Njg5MDA5NA==&mid=2247486854&idx=1&sn=fd77a6225b898b69c4f0e1a7e66cf105&chksm=ebd74c87dca0c5910a923a443ea6f694dd554b68df8cc00506570555b9cf7718a2ef2a058754&token=1109491988&lang=zh_CN#rd)
- [76页的集合](https://mp.weixin.qq.com/s?__biz=MzI4Njg5MDA5NA==&mid=2247486873&idx=1&sn=ce0752f481336ffba9b8f44265b2550e&chksm=ebd74c98dca0c58ee04162d7e5d07fd36c8ec1b32460a8a2396168a9fc5885a208810f0916f2&token=1109491988&lang=zh_CN#rd)
- [64页的JDBC](https://mp.weixin.qq.com/s?__biz=MzI4Njg5MDA5NA==&mid=2247486905&idx=1&sn=67fcd0558cfbdf6cd36de98cbd93afaf&chksm=ebd74cb8dca0c5ae052e6d216ed13458a9a17fa1b0f245bf740379b1d4b04b4e55fcbfb5adb4&token=1109491988&lang=zh_CN#rd)
- [105页的数据结构和算法](https://mp.weixin.qq.com/s?__biz=MzI4Njg5MDA5NA==&mid=2247486831&idx=1&sn=0d4b05e10d66eda1129f43348a8e3952&chksm=ebd74c6edca0c5786a5109a131d0501ef6bd02077e5ce1ad75d906cf3612a320d1098163e2d0&token=1109491988&lang=zh_CN#rd)
- [142页的Spring](https://mp.weixin.qq.com/s?__biz=MzI4Njg5MDA5NA==&mid=2247487013&idx=1&sn=f0d8c292738eb49bcd09cb2f6458dc69&chksm=ebd74f24dca0c632fa3ef8f205a2dd5c96531f78a68eae805e15b84de0b59774196a188aed14&token=306734573&lang=zh_CN#rd)
- [58页的过滤器和监听器](https://mp.weixin.qq.com/s?__biz=MzI4Njg5MDA5NA==&mid=2247487054&idx=1&sn=25f92798050d092027931e2ae0379e90&chksm=ebd74f4fdca0c6595bc795fd00354cf683d4593550cdd38ba7103893dd622606fc8f55fe6631&token=306734573&lang=zh_CN#rd)
- [30页的HTTP](https://mp.weixin.qq.com/s?__biz=MzI4Njg5MDA5NA==&mid=2247487071&idx=1&sn=511459730b3114fc77c60b82b54159b8&chksm=ebd74f5edca0c648c9b0c572fbc7fdbc26179ddcd7c45de37c06b6a2906a8ffc207f145b66f2&token=480724592&lang=zh_CN#rd)
- Hibernate
- AJAX
- Redis
- ......



#### 涵盖Java后端所有知识点的开源项目（已有8K+ star）：

- [GitHub](https://github.com/ZhongFuCheng3y/3y)
- [Gitee访问更快](https://gitee.com/zhongfucheng/Java3y)



如果大家想要**实时**关注我更新的文章以及分享的干货的话，微信搜索**Java3y**。



PDF文档的内容**均为手打**，有任何的不懂都可以直接**来问我**（公众号有我的联系方式）。

![](https://tva1.sinaimg.cn/large/00831rSTly1gctsejip0mj31ks0u04qp.jpg)

![](https://tva1.sinaimg.cn/large/00831rSTly1gdk64nngcnj31la0u0e5y.jpg)



![](https://tva1.sinaimg.cn/large/006tNbRwly1gb0nzpn8z7g30go0gokbp.gif)



**给三歪点个赞，对三歪真的非常重要！**