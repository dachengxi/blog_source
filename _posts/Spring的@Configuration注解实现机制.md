---
title: Spring的@Configuration注解实现机制
date: 2019-05-30 15:26:17
categories: Spring
tags:
	- Spring
---

记录下Spring中@Configuration注解的实现原理。

<!--more-->

@Configuration注解其实是@Component注解，容器启动过程中会在AnnotationConfigBeanDefinitionParser或者ComponentScanBeanDefinitionParser中注册ConfigurationClassPostProcessor，这个Processor就是用来处理@Configuration注解的。

ConfigurationClassPostProcessor实现了BeanDefinitionRegistryPostProcessor接口，并且优先级是最高的，容器刷新的时候会有一步是invokeBeanFactoryPostProcessor，这里面会调用ConfigurationClassPostProcessor的postProcessBeanDefinitionRegistry方法。这里会将@Configuration标注的类中的Bean扫描进容器中，注册成BeanDefinition。被@Configuration注解的类在ComponentScanBeanDefinitionParser扫描注解的时候已经被注册到容器中。这一步还会处理@Import、@ImportResource等等注解，将这些要导入的Bean注册到容器中。

// TODO

在postProcessBeanDefinitionRegistry方法处理完后，invokeBeanFactoryPostProcessor这一步会继续执行postProcessBeanFactory方法，这里会把@Configuration注解的类使用CGLIB进行增强。增强的作用主要是用来处理一个被@Bean注解的方法调用另外一个@Bean注解的方法的逻辑，如果被调用方法是@Bean注解的，则可以根据情况从容器中获取对应Bean。

增强处理完后，对于@Configuration和@Bean等注解的处理就完成了，后面就是Bean的实例化，实例化的时候会根据实际调用来使用增强中的方法进行处理。