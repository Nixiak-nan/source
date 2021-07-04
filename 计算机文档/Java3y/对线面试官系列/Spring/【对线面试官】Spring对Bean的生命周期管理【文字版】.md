3y：淦，已经熬到了第13面了

3y：上次问了Spring的基础

3y：IOC 和AOP基础都是烂大街的问题了

3y：上次没问原理，我估计这次会问

3y：问题不大好吧，这些题我在校招的时候就会了





面试官：上一面结束后，很多同学让我问Spring Bean的生命周期

面试官：这确实是一个比较好的面试题

面试官：很多应聘者觉得这些就都是背诵，能有什么用，八股文！

面试官：其实不然，作为Java后端对Spring可以说是重度依赖了

面试官：理解Spring Bean的生命周期可以让你更好地掌控自己的项目

面试官：不相信吗？看下去





3y：面试官你好，请问面试可以开始了吗？

面试官：嗯，开始吧

面试官：上周你讲完了Spring IOC，说Spring 对Bean还有一整套的生命周期管理

面试官：今天要不来聊聊Spring对Bean的生命周期管理？

3y：嗯，没问题的。

3y：很早之前我就看过源码，但Spring源码的实现类都太长了

3y：我也记不得很清楚某些实现类的名字

3y：要不我大概来说下流程？

面试官：没事，你开始吧



3y：首先要知道的是

3y：普通Java对象和Spring所管理的Bean实例化的过程是有些区别的

3y：在普通Java环境下创建对象简要的步骤可以分为：

3y：1):java源码被编译为被编译为class文件

3y：2):等到类需要被初始化时（比如说new、反射等）

3y：3):class文件被虚拟机通过类加载器加载到JVM

3y：4):初始化对象供我们使用

3y：简单来说，可以理解为它是用Class对象作为「模板」进而创建出具体的实例

面试官：....

3y：而Spring所管理的Bean不同的是，除了Class对象之外，还会使用BeanDefinition的实例来描述对象的信息

3y：比如说，我们可以在Spring所管理的Bean有一系列的描述：@Scope、@Lazy、@DependsOn等等

3y：可以理解为：Class只描述了类的信息，而BeanDefinition描述了对象的信息

面试官：嗯，这我大致了解你的意思了。

面试官：你就是想告诉我，Spring有BeanDefinition来存储着我们日常给Spring Bean定义的元数据（@Scope、@Lazy、@DependsOn等等），对吧？

3y：不愧是你

面试官：赶紧的，继续吧

3y：Spring在启动的时候需要「扫描」在XML/注解/JavaConfig 中需要被Spring管理的Bean信息

3y：随后，会将这些信息封装成BeanDefinition，最后会把这些信息放到一个beanDefinitionMap中

3y：我记得这个Map的key应该是beanName，value则是BeanDefinition对象

3y：到这里其实就是把定义的元数据加载起来，目前真实对象还没实例化

3y：接着会遍历这个beanDefinitionMap，执行BeanFactoryPostProcessor这个Bean工厂后置处理器的逻辑

3y：比如说，我们平时定义的占位符信息，就是通过BeanFactoryPostProcessor的子类PropertyPlaceholderConfigurer进行注入进去

3y：当然了，这里我们也可以自定义BeanFactoryPostProcessor来对我们定义好的Bean元数据进行获取或者需改

3y：只是一般我们不会这样干，实际上也很有少的使用场景

面试官：....

3y：BeanFactoryPostProcessor后置处理器执行完了以后，就到了实例化对象啦

3y：在Spring里边是通过反射来实现的，一般情况下会通过反射选择合适的构造器来把对象实例化

3y：但这里把对象实例化，只是把对象给创建出来，而对象具体的属性是还没注入的。

3y：比如我的对象是UserService，而UserService对象依赖着SendService对象，这时候的SendService还是null的

3y：所以，下一步就是把对象的相关属性给注入（：

3y：相关属性注入完之后，往下接着就是初始化的工作了

3y：首先判断该Bean是否实现了Aware相关的接口，如果存在则填充相关的资源

3y：比如我这边在项目用到的：我希望通过代码程序的方式去获取指定的Spring Bean

3y：我们这边会抽取成一个工具类，去实现ApplicationContextAware接口，来获取ApplicationContext对象进而获取Spring  Bean

3y：Aware相关的接口处理完之后，就会到BeanPostProcessor后置处理器啦

3y：BeanPostProcessor后置处理器有两个方法，一个是before，一个是after（那肯定是before先执行、after后执行）

3y：这个BeanPostProcessor后置处理器是AOP实现的关键（关键子类AnnotationAwareAspectJAutoProxyCreator）

3y：所以，执行完Aware相关的接口就会执行BeanPostProcessor相关子类的before方法

3y：BeanPostProcessor相关子类的before方法执行完，则执行init相关的方法，比如说@PostConstruct、实现了InitializingBean接口、定义的init-method方法

3y：当时我还去官网去看他们的被调用「执行顺序」分别是：@PostConstruct、实现了InitializingBean接口以及init-method方法

3y：这些都是Spring给我们的「扩展」，像@PostConstruct我就经常用到

3y：比如说：对象实例化后，我要做些初始化的相关工作或者就启个线程去Kafka拉取数据

面试官：...

3y：等到init方法执行完之后，就会执行BeanPostProcessor的after方法

3y：基本重要的流程已经走完了，我们就可以获取到对象去使用了

3y：销毁的时候就看有没有配置相关的destroy方法，执行就完事了

面试官：嗯，了解

面试官：但我的观众好像不太满意

面试官：总感觉少了些什么

面试官：你看过Spring是怎么解决循环依赖的吗？

面试官：如果现在有个A对象，它的属性是B对象，而B对象的属性也是A对象

面试官：说白了就是A依赖B，而B又依赖A，Spring是怎么做的？

3y：嗯，这块我也是看过的，其实也是在Spring的生命周期里面嘛

3y：从上面我们可以知道，对象属性的注入在对象实例化之后的嘛。

3y：它的大致过程是这样的：

3y：首先A对象实例化，然后对属性进行注入，发现依赖B对象

3y：B对象此时还没创建出来，所以转头去实例化B对象

3y：B对象实例化之后，发现需要依赖A对象，那A对象已经实例化了嘛，所以B对象最终能完成创建

3y：B对象返回到A对象的属性注入的方法上，A对象最终完成创建

3y：这是大致的过程；

面试官：听起来你还会原理哦？

3y：Absolutely

3y：至于原理，其实就是用到了三级的缓存

3y：所谓的三级缓存其实就是三个Map...首先明确一定，我对这里的三级缓存定义是这样的：

3y：singletonObjects（一级，日常实际获取Bean的地方）；

3y：earlySingletonObjects（二级，还没进行属性注入，由三级缓存放进来）；

3y：singletonFactories（三级，Value是一个对象工厂）；

3y：再回到刚才讲述的过程中，A对象实例化之后，属性注入之前，其实会把A对象放入三级缓存中

3y：key是BeanName，Value是ObjectFactory

3y：等到A对象属性注入时，发现依赖B，又去实例化B时

3y：B属性注入需要去获取A对象，这里就是从三级缓存里拿出ObjectFactory，从ObjectFactory得到对应的Bean（就是对象A）

3y：把三级缓存的A记录给干掉，然后放到二级缓存中

3y：显然，二级缓存存储的key是BeanName，value就是Bean（这里的Bean还没做完属性注入相关的工作）

3y：等到完全初始化之后，就会把二级缓存给remove掉，塞到一级缓存中

3y：我们自己去getBean的时候，实际上拿到的是一级缓存的

3y：大致的过程就是这样

面试官：那我想问一下，为什么是三级缓存？

3y：首先从第三级缓存说起（就是key是BeanName，Value为ObjectFactory）

3y：我们的对象是单例的，有可能A对象依赖的B对象是有AOP的（B对象需要代理）

3y：假设没有第三级缓存，只有第二级缓存（Value存对象，而不是工厂对象）

3y：那如果有AOP的情况下，岂不是在存入第二级缓存之前都需要先去做AOP代理？这不合适嘛

3y：这里肯定是需要考虑代理的情况的，比如A对象是一个被AOP增量的对象，B依赖A时，得到的A肯定是代理对象的

3y：所以，三级缓存的Value是ObjectFactory，可以从里边拿到代理对象

3y：而二级缓存存在的必要就是为了性能，从三级缓存的工厂里创建出对象，再扔到二级缓存（这样就不用每次都要从工厂里拿）

3y：应该很好懂吧？

面试官：确实（：

3y：我稍微总结一下今天的内容吧

3y：怕你的观众说不满意，那我就没有赞了，没有赞我就很难受

3y：首先是Spring Bean的生命周期过程，Spring使用BeanDefinition来装载着我们给Bean定义的元数据

3y：实例化Bean的时候实际上就是遍历BeanDefinitionMap

3y：Spring的Bean实例化和属性赋值是分开两步来做的

3y：在Spring Bean的生命周期，Spring预留了很多的hook给我们去扩展

3y：1）：Bean实例化之前有BeanFactoryPostProcessor

3y：2）：Bean实例化之后，初始化时，有相关的Aware接口供我们去拿到Context相关信息

3y：3）：环绕着初始化阶段，有BeanPostProcessor（AOP的关键）

3y：4）：在初始化阶段，有各种的init方法供我们去自定义

3y：而循环依赖的解决主要通过三级的缓存

3y：在实例化后，会把自己扔到三级缓存（此时的key是BeanName，Value是ObjectFactory）

3y：在注入属性时，发现需要依赖B，也会走B的实例化过程，B属性注入依赖A，从三级缓存找到A

3y：删掉三级缓存，放到二级缓存

面试官：嗯，你要不后面放点关键的源码吧

3y：这你倒是提醒我了，确实有必要

面试官：这要是能听懂，是真的看过源码才行（：

面试官：还好我看过



关键源码方法（强烈建议自己去撸一遍）

- `org.springframework.context.support.AbstractApplicationContext#refresh`(入口)
- `org.springframework.context.support.AbstractApplicationContext#finishBeanFactoryInitialization`(初始化单例对象入口)
- `org.springframework.beans.factory.config.ConfigurableListableBeanFactory#preInstantiateSingletons`(初始化单例对象入口)
- `org.springframework.beans.factory.support.AbstractBeanFactory#getBean(java.lang.String)`（万恶之源，获取并创建Bean的入口）
- `org.springframework.beans.factory.support.AbstractBeanFactory#doGetBean`（实际的获取并创建Bean的实现）
- `org.springframework.beans.factory.support.DefaultSingletonBeanRegistry#getSingleton(java.lang.String)`（从缓存中尝试获取）
- `org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#createBean(java.lang.String, org.springframework.beans.factory.support.RootBeanDefinition, java.lang.Object[])`（实例化Bean）
- `org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#doCreateBean`（实例化Bean具体实现）
- `org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#createBeanInstance`（具体实例化过程）
- `org.springframework.beans.factory.support.DefaultSingletonBeanRegistry#addSingletonFactory`（将实例化后的Bean添加到三级缓存）
- `org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#populateBean`（实例化后属性注入）
- `org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#initializeBean(java.lang.String, java.lang.Object, org.springframework.beans.factory.support.RootBeanDefinition)`（初始化入口）



![](https://tva1.sinaimg.cn/large/008eGmZEly1gne0dib6jwj30u10g50xh.jpg)

![](https://tva1.sinaimg.cn/large/008eGmZEly1gne0ey29w2j30tt0cqtdy.jpg)

![来源：https://www.jianshu.com/p/6c359768b1dc](https://tva1.sinaimg.cn/large/008eGmZEly1gne0b6gfbpj30vd0u0wje.jpg)

**文章以纯面试的角度去讲解，所以有很多的细节是未铺垫的。**

鉴于很多同学反馈没看懂【**对线面试官**】系列，基础相关的知识我确实写过文章讲解过啦，但有的同学就是不爱去翻。

为了让大家有更好的体验，我把基础文章也找出来（重要的知识点我还整理过**电子书**，比如说像多线程、集合这种面试必考的早就已经转成PDF格式啦）

我把这些**上传到网盘**，你们有需要直接下载就好了。做到这份上了，**不会还想白嫖吧**？**点赞**和**转发**又不用钱。

![](https://tva1.sinaimg.cn/large/008eGmZEly1gmisu5stxtj315w0du0to.jpg)



链接:[https://pan.baidu.com/s/1pQTuKBYsHLsUR5ORRAnwFg](https://pan.baidu.com/s/1pQTuKBYsHLsUR5ORRAnwFg)  密码:3wom



欢迎关注我的微信公众号【**Java3y**】来聊聊Java面试

<img src='https://tva1.sinaimg.cn/large/006tNbRwly1gb0nzpn8z7g30go0gokbp.gif' width=200px height=200px />

**[【对线面试官】系列](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzU4NzA3MTc5Mg==&action=getalbum&album_id=1657204970858872832#wechat_redirect) 一周两篇持续更新中！**





