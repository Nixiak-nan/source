面试官：来讲讲什么是注解吧

三歪：注解在我的理解下，就是代码中的特殊标记，这些标记可以在编译、类加载、运行时被读取，并执行相对应的处理。

面试官：你这讲得有点抽象，你先说说你在开发中有没有用到注解吧。

三歪：注解其实在开发中是非常常见的，比如我们在使用各种框架时（像我们Java程序员接触最多的还是Spring框架一套），就会用到非常多的注解，@Controller /  @Param / @Select 等等。一些项目也用到lombok的注解，@Slf4j / @Data 等等

三歪：除了框架实现的注解，Java原生也有@Overried、@Deprecated、@FunctionalInterface等基本注解。Java原生的基本注解大多数用于「标记」和「检查」

三歪：原生Java除了这些提供基本注解之外，还有一种叫做元Annotation（元注解），所谓的元Annotation就是用来修饰注解的。常用的元Annotation有@Retention 和@Target。@Retention注解可以简单理解为设置注解的生命周期，而@Target表示这个注解可以修饰哪些地方（比如方法、还是成员变量、还是包等等）

面试官：嗯，听得出来你在工作中是真的用到注解的，这些可能是Spring、Mybatis等框架原生提供的，可能是Java原生的。那你自己写过注解吗？

三歪：嗯，写过的。背景是这样的：我司有个监控告警系统，对外提供了客户端供我们自己使用。监控一般的指标就是QPS、RT和错误嘛。

三歪：原生的客户端需要在代码里指定上报，这会导致这种监控的代码会跟业务代码混合，比较恶心。

```java
public void send(String userName) {
  try {
    // qps 上报
    qps(params);
    long startTime = System.currentTimeMillis();

    // 构建上下文(模拟业务代码）
    ProcessContext processContext = new ProcessContext();
    UserModel userModel = new UserModel();
    userModel.setAge("22");
    userModel.setName(userName);
    //...

    // rt 上报
    long endTime = System.currentTimeMillis();
    rt(endTime - startTime);
  } catch (Exception e) {
    
    // 出错上报
    error(params);
  }
}
```

三歪：其实这种基础的监控信息，显然都可以通过AOP切面的方式去处理掉（可以看到都是方法级的）。而再用注解这个载体配置相关的信息，配合AOP解析就会比较优雅

面试官：👍可以的。

三歪：接下来讲讲我是怎么做的吧。

三歪：要写自定义的注解，首先考虑我们是在什么时候解析这个注解。这就需要用到前面所说的@Retention注解，这个注解会修饰我们自定义注解生命周期。

三歪：@Retention注解传入的是RetentionPolicy枚举，该枚举有三个常量，分别是SOURCE、CLASS和RUNTIME

三歪：SOURCE代表着注解仅保留在源级别中，并由编译器忽略。CLASS代表着注解在编译时由编译器保留，但Java虚拟机（JVM）会忽略。RUNTIME代表着标记的注解会由JVM保留，因此运行时环境可以使用它。

三歪：理解这块就得了解从 .java 文件到 class 文件再到 class 被jvm加载的过程了。下面的图描述着从.java文件到编译为class文件的过程

![](https://tva1.sinaimg.cn/large/0081Kckwly1glt5yhqk8oj31180ag4bs.jpg)

三歪：从上面的图可以发现有个「注解抽象语法树」，这里其实就会去解析注解，然后做处理的逻辑。

三歪：所以重点来了，如果你想要在编译期间处理注解相关的逻辑，你需要继承AbstractProcessor 并实现process方法。比如可以看到lombok就用AnnotationProcessor继承了AbstractProcessor。

三歪：一般来说，只要自定义的注解中@Retention注解设置为SOURCE和CLASS这俩个级别，那么就需要继承并实现（因为SOURCE和CLASS这俩个级别等加载到jvm的时候，注解就被抹除了）

三歪：从这里又引申出：lombok的实现原理就是在这（为什么使用了个@Data这样的注解就能有set/get等方法了，就是在这里加上去的）

面试官：嗯，你是还有点东西哦

三歪：一般来说，我们自己定义的注解都是RUNTIME级别的，因为大多数情况我们是根据运行时环境去做一些处理。

三歪：我们现实在开发的过程中写自定义注解需要配合反射来使用（因为反射是Java获取运行时的信息的重要手段）。

三歪：所以，我当时就用了自定义注解，在Spring AOP的逻辑处理中，判断是否带有自定义注解，如果有则将监控的逻辑写在方法的前后。这样，只要在方法上加上我的注解，那就可以有对方法监控的效果（RT、QPS、ERROR）

```java
@Around("@annotation(com.sanwai.service.openapi.monitor.Monitor)")
    public Object antispan(ProceedingJoinPoint pjp) throws Throwable {

        String functionName = pjp.getSignature().getName();
        Map<String, String> tags = new HashMap<>();

        logger.info(functionName);

        tags.put("functionName", functionName);
        tags.put("flag", "done");

        monitor.sum(functionName, "start", 1);

        //方法执行开始时间
        long startTime = System.currentTimeMillis();

        Object o = null;
        try {
            o = pjp.proceed();
        } catch (Exception e) {
            //方法执行结束时间
            long endTime = System.currentTimeMillis();

            tags.put("flag", "fail");
            monitor.avg("rt", tags, endTime - startTime);

            monitor.sum(functionName, "fail", 1);
            throw e;
        }

        //方法执行结束时间
        long endTime = System.currentTimeMillis();

        monitor.avg("rt", tags, endTime - startTime);

        if (null != o) {
            monitor.sum(functionName, "done", 1);
        }
        return o;
    }
```

面试官：嗯，总体来看，你对注解这块基础还是扎实的。你当时是从哪里学这块的啊，挺不错的



