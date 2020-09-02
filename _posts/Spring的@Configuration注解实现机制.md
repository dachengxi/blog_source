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

ConfigurationClassPostProcessor实现了BeanDefinitionRegistryPostProcessor接口，并且优先级是最高的，容器刷新的时候会有一步是invokeBeanFactoryPostProcessor，这里面会调用ConfigurationClassPostProcessor的postProcessBeanDefinitionRegistry方法。