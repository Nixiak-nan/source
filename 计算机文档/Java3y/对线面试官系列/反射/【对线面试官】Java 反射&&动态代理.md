![](https://tva1.sinaimg.cn/large/0081Kckwly1gmd4itqsxdj30ku11279l.jpg)

![](https://tva1.sinaimg.cn/large/0081Kckwly1gmd4n5k0g7j30ku112jwa.jpg)

![](https://tva1.sinaimg.cn/large/0081Kckwly1gmd4osmjw9j30ku1127b6.jpg)

![](https://tva1.sinaimg.cn/large/0081Kckwly1gmd4q0xp5gj30ku112107.jpg)

![](https://tva1.sinaimg.cn/large/0081Kckwly1gmd4rh2lk6j30ku112ahj.jpg)

![](https://tva1.sinaimg.cn/large/0081Kckwly1gmd4so3iiwj30ku112wku.jpg)

![](https://tva1.sinaimg.cn/large/0081Kckwly1gmd4u7wh5uj30ku112dmf.jpg)



![](https://tva1.sinaimg.cn/large/0081Kckwly1gmd4y3vvazj30ku112q9a.jpg)

```java
// 抽象类，定义泛型<T>
public abstract class BaseDao<T> {
    public BaseDao(){
        Class clazz = this.getClass();
        ParameterizedType  pt = (ParameterizedType) clazz.getGenericSuperclass(); 
        clazz = (Class) pt.getActualTypeArguments()[0];
        System.out.println(clazz);
    }
}

// 实现类
public class UserDao extends BaseDao<User> {
    public static void main(String[] args) {
        BaseDao<User> userDao = new UserDao();

    }
}
// 执行结果输出
class com.entity.User
```

![](https://tva1.sinaimg.cn/large/0081Kckwly1gmd50xxey2j30ku112tff.jpg)

![](https://tva1.sinaimg.cn/large/0081Kckwly1gmd53eggtqj30ku112gs7.jpg)

![](https://tva1.sinaimg.cn/large/0081Kckwly1gmd553n03ij30ku112qa0.jpg)

![](https://tva1.sinaimg.cn/large/0081Kckwly1gmd888sjbtj30ku112grh.jpg)

**文章以纯面试的角度去讲解，所以有很多的细节是未铺垫的。**

比如说反射的API使用以及动态代理的详解等等等...这些在【**Java3y**】都有过详细的基本教程甚至电子书，我就不再详述了。



过了一天，面试官看大家三连了。又给我补充了道题**：都说反射会影响性能，有什么方式可以减低它的性能影响吗？**



答案：可以使用缓存把反射的元数据存储起来，下一次使用的时候就可以直接从内存获取了。尽可能使用**高性能的反射框架**（都帮你封装好了，不用自己实现）



面试官补问：有什么**高性能的反射框架**推荐吗？

