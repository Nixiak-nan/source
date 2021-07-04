![](https://tva1.sinaimg.cn/large/008eGmZEly1gngdx3irp0j30ku112qak.jpg)

![](https://tva1.sinaimg.cn/large/008eGmZEly1gngdwigtwgj30ku112jxl.jpg)

![](https://tva1.sinaimg.cn/large/008eGmZEly1gngdwxfejfj30ku112wj7.jpg)

![](https://tva1.sinaimg.cn/large/008eGmZEly1gngdwupi1uj30ku1120yp.jpg)

![](https://tva1.sinaimg.cn/large/008eGmZEly1gngdwmsq8uj30ku112dl9.jpg)

![](https://tva1.sinaimg.cn/large/008eGmZEly1gngdwst87rj30ku112gr9.jpg)

![](https://tva1.sinaimg.cn/large/008eGmZEly1gngdwixmc0j30ku112tfq.jpg)

![](https://tva1.sinaimg.cn/large/008eGmZEly1gngdwhfi89j30ku1120yf.jpg)

![](https://tva1.sinaimg.cn/large/008eGmZEly1gngdww13f4j30ku1120z4.jpg)

![](https://tva1.sinaimg.cn/large/008eGmZEly1gngdwqf5krj30ku112n3n.jpg)

![](https://tva1.sinaimg.cn/large/008eGmZEly1gngdwojjtjj30ku11244y.jpg)

![](https://tva1.sinaimg.cn/large/008eGmZEly1gngdwx195yj30ku112dmh.jpg)

![](https://tva1.sinaimg.cn/large/008eGmZEly1gngdwu5m5kj30ku112q93.jpg)

![](https://tva1.sinaimg.cn/large/008eGmZEly1gngdwtgd61j30ku11243t.jpg)

![](https://tva1.sinaimg.cn/large/008eGmZEly1gngdwnqolrj30ku1127ag.jpg)

![](https://tva1.sinaimg.cn/large/008eGmZEly1gngdwxwpq0j30ku112dlw.jpg)

![](https://tva1.sinaimg.cn/large/008eGmZEly1gngdy2uszpj30ku112wks.jpg)

![](https://tva1.sinaimg.cn/large/008eGmZEly1gngdwrtyqcj30ku11244o.jpg)

![](https://tva1.sinaimg.cn/large/008eGmZEly1gngdwwj5ehj30ku112q8x.jpg)

![](https://tva1.sinaimg.cn/large/008eGmZEly1gngdwv9x09j30ku11245q.jpg)

![](https://tva1.sinaimg.cn/large/008eGmZEly1gngdwjzcrxj30ku112aga.jpg)

![](https://tva1.sinaimg.cn/large/008eGmZEly1gngdwpy4lhj30ku112q8z.jpg)

![](https://tva1.sinaimg.cn/large/008eGmZEly1gngdwr0fyoj30ku112jxo.jpg)

![](https://tva1.sinaimg.cn/large/008eGmZEly1gngdwp8042j30ku1127b1.jpg)

![](https://tva1.sinaimg.cn/large/008eGmZEly1gngdwliqgzj30ku112dmp.jpg)

![](https://tva1.sinaimg.cn/large/008eGmZEly1gngdyl1dhej30ku112dk4.jpg)



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



去网上看博客的时候，找到了几张比较好的图，这里贴下方便大家理解吧~



![](https://tva1.sinaimg.cn/large/008eGmZEly1gne0dib6jwj30u10g50xh.jpg)

![](https://tva1.sinaimg.cn/large/008eGmZEly1gne0ey29w2j30tt0cqtdy.jpg)

![来源：https://www.jianshu.com/p/6c359768b1dc](https://tva1.sinaimg.cn/large/008eGmZEly1gne0b6gfbpj30vd0u0wje.jpg)



