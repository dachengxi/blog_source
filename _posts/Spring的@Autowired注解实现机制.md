---
title: SpringBoot自动配置的原理
date: 2019-05-26 13:48:02
categories: Spring
tags:
	- Spring
---

记录下Spring中@Autowired注解的实现原理。

<!--more-->

@Autowired注解使用的方式有如下几种：

- setter方法
- 构造方法
- 成员变量
- 普通方法

注解的激活方式有使用`<annotation-config/>`或者`<component-scan/>`。这两个标签就是入口。

@Autowired注解解析过程简单描述如下：

- 遇到annotation-config标签的时候，使用ContextNamespaceHandler来注册一个对应的处理器：AnnotationConfigBeanDefinitionParser
- AnnotationConfigBeanDefinitionParser中注册一堆的PostProcessor，其中包括：@Configuration、@Autowired、@Required、@Resource、@PostConstruct等等注解的PostProcessor
- 这些PostProcessor会在不同的阶段被触发，用来处理对应的注解。@Autowired注解使用AutowiredAnnotationBeanPostProcessor来解析。

当Spring解析配置文件为BeanDefinition的时候遇到annotation-config标签，会使用ContextNamespaceHandler来处理，这个Handler中注册了很多的BeanDefinitionParser。其中annotation-config对应的解析器是AnnotationConfigBeanDefinitionParser。

AnnotationConfigBeanDefinitionParser中首先注册一些注解的处理器，这些处理器都是各种类型的BeanPostProcessor，这些BeanPostProcessor后面会各自处理不同的注解。其中Autowired注解处理器是：AutowiredAnnotationBeanPostProcessor。

AutowiredAnnotationBeanPostProcessor实例化的时候，会先将Autowired、Value、Inject等注解一个Set中保存起来。AutowiredAnnotationBeanPostProcessor继承了InstantiationAwareBeanPostProcessorAdapter，会在实例化Bean的时候被调用，用来查找Bean的构造器，方法是：determineCandidateConstructors

