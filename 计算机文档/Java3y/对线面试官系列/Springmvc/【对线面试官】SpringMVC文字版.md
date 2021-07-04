3y：淦，已经熬到了第10面了

3y：不知道面试官会问我些什么问题呢

3y：最近开发有点忙，都没什么时间了

3y：哎，有点小慌



3y：面试官你好，请问面试可以开始了吗？

面试官：嗯，开始吧

面试官：今天要不来聊聊SpringMVC吧？

3y：基础不问了吗？多线程我看还有好多知识点

面试官：你管我面什么，你答不答？

面试官：你不也看看多线程的阅读有多么凄惨？

面试官：评论都喊着要看Spring家族呢

3y：好吧...

3y：我先简单说下我对SpringMVC的理解哈

3y：SpringMVC我觉得它是对Servlet的封装，屏蔽掉Servlet很多的细节

3y：举几个例子

3y：可能我们刚学Servlet的时候，要获取参数需要不断的getParameter

3y：现在只要在SpringMVC方法定义对应的JavaBean，只要属性名与参数名一致，SpringMVC就可以帮我们实现「将参数封装到JavaBean」上了

3y：又比如，以前使用Servlet「上传文件」，需要处理各种细节，写一大堆处理的逻辑（还得导入对应的jar）

3y：现在一个在SpringMVC的方法上定义出MultipartFile接口，又可以屏蔽掉上传文件的细节了。

3y：例子还有很多，我就不一一赘述了。



面试官：既然你说SpringMVC是对Servlet的封装，你了解SpringMVC请求处理的流程吗？

3y：嗯，当然了，我看过源码。总体流程大概是这样的

3y：1):首先有个统一处理请求的入口

3y：2):随后根据请求路径找到对应的映射器

3y：3):找到处理请求的适配器

3y：4):拦截器前置处理

3y：5):真实处理请求（也就是调用真正的代码)

3y：6): 视图解析器处理

3y：7):拦截器后置处理

面试官：嗯，了解，可以再稍微深入点吗？

面试官：毕竟这随便在网上找张SpringMVC流程图，就可以答出来了

面试官：看不出来你看过源码啊

3y：哦？

3y：那我就简单补充下细节吧。

3y：统一的处理入口，对应SpringMVC下的源码是在DispatcherServlet下实现的

3y：该对象在初始化就会把映射器、适配器、视图解析器、异常处理器、文件处理器等等给初始化掉

3y：至于会初始化哪些具体实例，看下DispatcherServlet.properties就知道了，都配置在那了

3y：所有的请求其实都会被doService方法处理，里边最主要就是调用doDispatch方法

3y：通过doDispatch方法我们就可以看到整个SpringMVC处理的流程

面试官：嗯

3y：查找映射器的时候实际就是找到「最佳匹配」的路径，具体方法实现我记得好像是在lookupHandlerMethod方法上

3y：从源码可以看到「查找映射器」实际返回的是HandlerExecutionChain，里边有映射器Handler+拦截器List

3y：前面提到的拦截器前置处理和后置处理就是用的HandlerExecutionChain中的拦截器List

面试官：嗯

3y：获取得到HandlerExecutionChain后，就会去获取适配器，一般我们获取得到的就是RequestMappingHandlerAdapter

3y：在代码里边可以看到的是，经常用到的@ResponseBody和@Requestbody的解析器

3y：就会在初始化的时候加到参数解析器List中

3y：得到适配器之后，就会执行拦截器前置处理

面试官：嗯

3y：拦截器前置处理执行完后，就会调用适配器对象实例的hanlde方法执行真正的代码逻辑处理

3y：核心的处理逻辑在invokeAndHandle方法中，会获取得到请求的参数并调用，处理返回值

3y：参数的封装以及处理会被适配器的参数解析器进行处理，具体的处理逻辑取决于HttpMessageConverter的实例对象

面试官：嗯，了解了。要不你再压缩下关键的信息

3y：DispatcherServlet（入口）->DispatcherServlet.properties（会初始化的对象）->HandlerMapping（映射器）->HandlerExecutionChain(映射器+拦截器List) ->HttpRequestHandlerAdapter(适配器)->HttpMessageConverter(数据转换)

面试官：最后来画张流程图吧？

3y：没问题





