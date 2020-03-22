---
title: Spring中Bean的生命周期总结
date: 2020-03-22 22:21:18
categories: Spring
tags:
	- Spring
	- Bean
---

总结一下Spring中Bean的生命周期，对于具体的源码分析以前写过文档，网上也有很多类似的文章，这里不做重复。

<!--more-->

Spring中Bean的生命周期或者说任何一个物体的生命周期都可以大体分为：初始化、使用、销毁，哪怕是Spring IOC容器也可以这样总结。这里说要总结生命周期，其实就是把这三步稍微详细的再说一下。大体的过程如下：

1. 手动或者自动的触发获取一个Bean，使用BeanFactory的时候需要我们代码自己获取Bean，ApplicationContext则是在IOC启动的时候自动初始化一个Bean。
2. IOC会根据BeanDefinition来实例化这个Bean，如果这个Bean还有依赖其他的Bean则会先初始化依赖的Bean，这里又涉及到了循环依赖的解决。实例化Bean的时候根据工厂方法、构造方法或者简单初始化等选择具体的实例来进行实例化，最终都是使用反射进行实例化。
3. Bean实例化完成，也就是一个对象实例化完成后，会继续填充这个Bean的各个属性，也是使用反射机制将属性设置到Bean中去。
4. 填充完属性后，会调用各种Aware方法，将需要的组件设置到当前Bean中。BeanFactory这种低级容器需要我们手动注册Aware接口，而ApplicationContext这种高级容器在IOC启动的时候就自动给我们注册了Aware等接口。
5. 接下来如果Bean实现了PostProcessor一系列的接口，会先调用其中的postProcessBeforeInitialization方法。BeanFactory这种低级容器需要我们手动注册PostProcessor接口，而ApplicationContext这种高级容器在IOC启动的时候就自动给我们注册了PostProcessor等接口。
6. 如果Bean实现了InitializingBean接口，则会调用对应的afterPropertiesSet方法。
7. 如果Bean设置了init-method属性，则会调用init-method指定的方法。
8. 接下来如果Bean实现了PostProcessor一系列的接口，会先调用其中的postProcessAfterInitialization方法。BeanFactory这种低级容器需要我们手动注册PostProcessor接口，而ApplicationContext这种高级容器在IOC启动的时候就自动给我们注册了PostProcessor等接口。
9. 到这里Bean就可以使用了。
10. 容器关闭的时候需要销毁Bean。
11. 如果Bean实现了DisposableBean，则调用destroy方法。
12. 如果Bean配置了destroy-method属性，则调用指定的destroy-method方法。

在实例化完Bean和填充属性之前还会涉及到AOP的处理，在处理依赖Bean的时候还会涉及到循环依赖的处理等等细节。