3y：淦，已经熬到了第11面了

3y：哎，也不知道能不能回家过年

3y：好早之前就买了机票了

3y：还未必能回得去...

3y：11面了，不知道这次会问些什么内容呢



3y：面试官你好，请问面试可以开始了吗？

面试官：嗯，开始吧

面试官：今天要不来聊聊ThreadLocal吧？

面试官：你简单来讲讲什么是ThreadLocal吧

3y：我个人对ThreadLocal理解就是

3y：它提供了线程的局部变量，每个线程都可以通过set/get来对这个局部变量进行操作

3y：不会和其他线程的局部变量进行冲突，实现了线程的数据隔离

面试官：你在工作中有用到过ThreadLocal吗？

3y：这块是真不多，不过还是有一处的。就是我们项目有个的DateUtils工具类

3y：这个工具类主要是对时间进行格式化

3y：格式化/转化的实现是用的SimpleDateFormat

3y：但众所周知SimpleDateFormat不是线程安全的，所以我们就用ThreadLocal来让每个线程装载着自己的SimpleDateFormat对象

3y：以达到在格式化时间时，线程安全的目的

3y：在方法上创建SimpleDateFormat对象也没问题，但每调用一次就创建一次有点不优雅

3y：在工作中ThreadLocal的应用场景确实不多，但要不我给你讲讲Spring是怎么用的？

面试官：好吧，你讲讲呗



3y：Spring提供了事务相关的操作，而我们知道事务是得保证一组操作同时成功或失败的

3y：这意味着我们一次事务的所有操作需要在同一个数据库连接上

3y：但是在我们日常写代码的时候是不需要关注这点的

3y：Spring就是用的ThreadLocal来实现，ThreadLocal存储的类型是一个Map

3y：Map中的key 是DataSource，value 是Connection（为了应对多数据源的情况，所以是一个Map）

3y：用了ThreadLocal保证了同一个线程获取一个Connection对象，从而保证一次事务的所有操作需要在同一个数据库连接上

面试官：了解



面试官：你知道ThreadLocal内存泄露这个知识点吗？

3y：怎么都喜欢问这个...

3y：了解的，要不我先来讲讲ThreadLocal的原理？

面试官：请开始你的表演吧

3y：ThreadLocal是一个壳子，真正的存储结构是ThreadLocal里有ThreadLocalMap这么个内部类

3y：而有趣的是，ThreadLocalMap的引用是在Thread上定义的

3y：ThreadLocal本身并不存储值，它只是作为key来让线程从ThreadLocalMap获取value

3y：所以，得出的结论就是ThreadLocalMap该结构本身就在Thread下定义，而ThreadLocal只是作为key，存储set到ThreadLocalMap的变量当然是线程私有的咯

面试官：那我想问下，我可以在ThreadLocal下定义Map，key是Thread，value是set进去的值吗？

面试官：就是说，为啥我要把ThreadLocal做为key，而不是Thread做为key？这样不是更清晰吗？

3y：嗯，我明白你的意思。

3y：理论上是可以，但没那么优雅。

3y：你提出的做法实际上就是所有的线程都访问ThreadLocal的Map，而key是当前线程

3y：但这有点小问题，一个线程是可以拥有多个私有变量的嘛，那key如果是当前线程的话，意味着还点做点「手脚」来唯一标识set进去的value

3y：假设上一步解决了，还有个问题就是；并发量足够大时，意味着所有的线程都去操作同一个Map，Map体积有可能会膨胀，导致访问性能的下降

3y：这个Map维护着所有的线程的私有变量，意味着你不知道什么时候可以「销毁」

3y：现在JDK实现的结构就不一样了。

3y：线程需要多个私有变量，那有多个ThreadLocal对象足以，对应的Map体积不会太大

3y：只要线程销毁了，ThreadLocalMap也会被销毁

面试官：嗯，了解。



面试官：回到ThreadLocal内存泄露上吧

面试官：谈谈你对这个的理解呗

3y：ThreadLocal内存泄露其实发生的概率非常非常低，我也不知道为什么这么喜欢问。

3y：回到原理上，我们知道Thread在创建的时候，会有栈引用指向Thread对象，Thread对象内部维护了ThreadLocalMap引用

3y：而ThreadLocalMap的Key是ThreadLocal，value是传入的Object

3y：ThreadLocal对象会被对应的栈引用关联，ThreadLocalMap的key也指向着ThreadLocal

3y：ThreadLocalRef && ThreadLocalMap Entry key ->ThreadLocal

3y：ThreadRef->Thread->ThreadLoalMap-> Entry value-> Object

3y：网上大多分析的是ThreadLocalMap的key是弱引用指向ThreadLocal

3y：首先我来讲讲什么叫做弱引用

3y：嗯...要不顺便讲讲Java的4种引用吧

面试官：嗯

3y：强引用是最常见的，只要把一个对象赋给一个引用变量，这个引用变量就是一个强引用

3y：强引用：只要对象没有被置null，在GC时就不会被回收

3y：软引用相对弱化了一些，需要继承 SoftReference实现

3y：软引用：如果内存充足，只有软引用指向的对象不会被回收。如果内存不足了，只有软引用指向的对象就会被回收

3y：弱引用又更弱了一些，需要继承WeakReference实现

3y：弱引用：只要发生GC，只有弱引用指向的对象就会被回收

3y：最后就是虚引用，需要继承PhantomReference实现

3y：虚引用的主要作用是：跟踪对象垃圾回收的状态，当回收时通过引用队列做些「通知类」的工作

3y：了解了这几种引用之后，再回过头来看ThreadLocal

面试官：嗯

3y：ThreadLocal内存泄露指的是：ThreadLocal被回收了，ThreadLocalMap Entry的key没有了指向

3y：但Entry仍然有ThreadRef->Thread->ThreadLoalMap-> Entry value-> Object 这条引用一直存在导致内存泄露

面试官：嗯

3y：为什么我说导致内存泄露的概率非常低呢，我觉得是这样的

3y：首先ThreadLocal被两种引用指向

3y：1):ThreadLocalRef->ThreadLocal（强引用）

3y：2):ThreadLocalMap Entry key ->ThreadLocal（弱引用）

3y：只要ThreadLocal没被回收（使用时强引用不置null），那ThreadLocalMap Entry key的指向就不会在GC时断开被回收，也没有内存泄露一说法

3y：通过ThreadLocal了解实现后，又知道ThreadLocalMap是依附在Thread上的，只要Thread销毁，那ThreadLocalMap也会销毁

3y：那非线程池环境下，也不会有长期性的内存泄露问题

3y：而ThreadLocal实现下还做了些”保护“措施，如果在操作ThreadLocal时，发现key为null，会将其清除掉

3y：所以，如果在线程池（线程复用）环境下，如果还会调用ThreadLocal的set/get/remove方法

3y：发现key为null会进行清除，不会有长期性的内存泄露问题

3y：那存在长期性内存泄露需要满足条件：ThreadLocal被回收&&线程被复用&&线程复用后不再调用ThreadLocal的set/get/remove方法

3y：使用ThreadLocal的最佳实践就是：用完了，手动remove掉。就像使用Lock加锁后，要记得解锁

面试官：嗯

面试官：那我想问下，为什么要将ThreadLocalMap的key设置为弱引用呢？强引用不香吗？

3y：外界是通过ThreadLocal来对ThreadLocalMap进行操作的，假设外界使用ThreadLocal的对象被置null了，那ThreadLocalMap的强引用指向ThreadLocal也毫无意义啊。

3y：弱引用反而可以预防大多数内存泄漏的情况

3y：毕竟被回收后，下一次调用set/get/remove时ThreadLocal内部会清除掉

面试官：我看网上有很多人说建议把ThreadLocal修饰为static，为什么？

3y：ThreadLocal能实现了线程的数据隔离，不在于它自己本身，而在于Thread的ThreadLocalMap

3y：所以，ThreadLocal可以只初始化一次，只分配一块存储空间就足以了，没必要作为成员变量多次被初始化。

面试官：最后想问个问题：什么叫做内存泄露？

3y：.....

3y：意思就是：你申请完内存后，你用完了但没有释放掉，你自己没法用，系统又没法回收。











