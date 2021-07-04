![](https://tva1.sinaimg.cn/large/0081Kckwly1glt9p2vmmzj30ku1120xw.jpg)

![](https://tva1.sinaimg.cn/large/0081Kckwly1gltih430osj30ku112agg.jpg)

![](https://tva1.sinaimg.cn/large/0081Kckwly1gltij8cq5oj30ku11244n.jpg)

![](https://tva1.sinaimg.cn/large/0081Kckwly1gltio3gam0j30ku112jwb.jpg)

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

![](https://tva1.sinaimg.cn/large/0081Kckwly1glta3h5t4lj30ku1120yj.jpg)

![](https://tva1.sinaimg.cn/large/0081Kckwly1glta3vd0x2j30ku11244t.jpg)

![](https://tva1.sinaimg.cn/large/0081Kckwly1glta3mpjikj31180agdhz.jpg)

![](https://tva1.sinaimg.cn/large/0081Kckwly1gltit4d0fcj30ku112n3a.jpg)

![](https://tva1.sinaimg.cn/large/0081Kckwly1gltivai5imj30ku112tf4.jpg)

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

![](https://tva1.sinaimg.cn/large/0081Kckwly1gltj6ikpnyj30ku112teu.jpg)

![](https://tva1.sinaimg.cn/large/0081Kckwly1gltj1bkv5pj30ku1120ye.jpg)

![](https://tva1.sinaimg.cn/large/0081Kckwly1glu62dcty8j30ku11279l.jpg)

![](https://tva1.sinaimg.cn/large/0081Kckwly1gluhchuklpj30ku112grv.jpg)

**文章以纯面试的角度去讲解，所以有很多的细节是未铺垫的。**

比如说反射、`.java文件`到jvm的过程、AOP是什么等等等...这些在【**Java3y**】都有过详细的基本教程甚至电子书，我就不再详述了。



欢迎关注我的微信公众号【**面试造火箭**】来聊聊Java面试

![](https://tva1.sinaimg.cn/large/0081Kckwly1gluiplup1xj3076076glt.jpg)