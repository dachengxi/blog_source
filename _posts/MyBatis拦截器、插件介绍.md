---
title: MyBatis拦截器、插件介绍
date: 2020-11-17 20:02:41
categories: MyBatis
tags: 
	- MyBatis
---

MyBatis拦截器、插件学习。

<!--more-->

MyBatis中的插件机制其实是个拦截器，能在某些方法执行前后进行处理。实际上和Java动态代理差不多，而MyBatis在底层实现上也是使用动态代理来实现自己的插件机制。

MyBatis支持在四个地方进行拦截：

- Executor，可对执行器的方法进行拦截
- StatementHandler，可对SQL语法构建等方法进行拦截
- ParameterHandler，可对参数处理等方法进行拦截
- ResultSetHandler，可对结果集处理等方法进行拦截

MyBatis支持对上面四个类中的方法进行拦截。

# 使用

MyBatis拦截器的使用和动态代理很像，大概的使用方法如下：

1. 实现`org.apache.ibatis.plugin.Interceptor`接口，在intercept方法中实现对某些具体的方法进行拦截处理
2. 在该类上使用`@Intercepts`和`@Signature`注解，来指明要拦截的类（Executor、StatementHandler、ParameterHandler、ResultSetHandler中的一个或多个）和具体方法（前面四个类中的方法）
3. 在`mybatis-config.xml`中配置上面实现的拦截器

大概的使用方法就如上面所说的那么简单，实际使用场景可以有很多，比如：自己可以拦截sql进行打印、时间统计、sql调用数据记录、实现分表逻辑等等。

# 实现

MyBatis插件/拦截器的实现，底层使用动态代理实现，一层一层的代理进行包装。实现大概分三部分：

- 对自定义拦截器的解析收集工作，在解析配置文件的时候进行
- 对拦截器进行插入，也就是动态代理的创建过程，分别在newExecutor、newStatementhandler、newResultSetHandler、newParameterHandler方法被调用的时候进行创建
- 具体使用过程中的调用

## 收集拦截器

在XMLConfigBuilder解析mybatis-config.xml配置文件的时候有一步是`pluginElement(root.evalNode("plugins"));`，这里面就是将配置文件中配置的plugin解析出来，并进行实例化，然后添加到InterceptorChain的interceptors列表中缓存起来。

## 拦截器动态代理创建

创建的时机如下：

- DefaultSqlSessionFactory创建SqlSession的时候，会调用Cofiguration的newExecutor方法创建Executor，这里面会执行Executor相应的拦截器的代理创建
- 各个Executor执行sql的时候，会先创建StatementHandler，调用的方法是Configuration的newStatementHandler，这里面会执行StatementHandler相关的拦截器的代理创建
- 在创建StatementHandler具体实例的时候，会创建ParameterHandler和ResultSetHandler对象，调用的是Configuration的newParameterHandler和newResultSetHandler方法，这里面会分别执行ParameterHandler的和ResultSetHandler的相关的拦截器的代理创建

代理的创建，就跟使用Java动态代理创建代理一样，上面已经收集了自定义的拦截器，并且缓存到了InterceptorChain中的interceptors列表中，这里就会使用InterceptorChain的pluginAll方法，将对应的Interceptor进行代理的创建，代理的创建最终是在Plugin的wrap方法中完成的。Plugin实现了InvocationHandler接口，这里就跟平时使用Java动态代理创建代理的方式基本一样了。

Plugin创建代理的时候，会解析自定义拦截器上的注解：`@Intercepts`和`@Signature`，将要拦截的类和方法缓存到signatureMap中，等到后面执行代理的时候，会先到signatureMap中查找要执行的方法是不是需要被拦截的，如果是，则先进行拦截器的执行，最后再进行实际方法的调用。

如果有多个拦截器要用到同一个目标对象上，这里使用的是嵌套代理，也就是一层套一层，最里面是目标对象的代理，往外一层是代理的代理、代理的代理的代理。这也是责任链模式。

## 拦截器代理的调用

目标对象已经被创建成了代理对象，并且代理对象中包含了对应的拦截器和要拦截的方法。执行目标对象调用，其实就是在执行对应的代理，执行代理的时候会先到signatureMap中查找要执行的方法是不是需要被拦截的，如果是，则先进行拦截器的执行，最后再进行实际方法的调用。

# 示例

使用MyBatis插件实现简易分表功能代码：[https://github.com/dachengxi/medlar/blob/master/spring-boot-mybatis/src/main/java/me/cxis/mybatis/pluginSplitTableInterceptor.java](https://github.com/dachengxi/medlar/blob/master/spring-boot-mybatis/src/main/java/me/cxis/mybatis/pluginSplitTableInterceptor.java)