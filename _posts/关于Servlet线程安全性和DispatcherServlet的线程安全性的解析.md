---
title: 关于Servlet线程安全性和DispatcherServlet的线程安全性的解析
date: 2017-04-13 16:33:01
categories: 
	- Java基础
	- Servlet
tags:
	- Servlet
---
我们知道在Servlet第一次被调用的时候，Servlet容器会根据web.xml中配置的信息去实例化Servlet，而且这个Servlet只会被实例化一次。当多个请求同时到来时，可能会使用同一个Servlet进行处理，这时候就会涉及到线程安全的问题。
<!--more-->
纠结！！！
# Servlet的线程安全性？不确定
Servlet是单实例多线程的方式来处理请求，这应该就是造成线程安全的主要原因了。我们知道Servlet本身是无状态的，也就是说Servlet本身是线程安全的，但是为什么网上都说Servlet是线程不安全的呢？可能就是根据一句多个线程会同时访问一个Servlet实例来判断的把。

而Servlet是不是线程安全的，主要是由实现来决定的，如果一个Servlet实现有实例变量，并且会被多线程更改，这时候就不是线程安全的；而如果有实例变量，但是变量又是只读的，这时候不涉及变量的更改，就是线程安全的；而如果一个Servlet实现没有实例变量，都是局部变量，这时候也是线程安全的。

## HttpServlet
HttpServlet是Servlet的一个实现，继承自GenericServlet，并且也是我们自定义Servlet所要继承的一个类，HttpServlet是不是线程安全的，也不好说，也得根据在使用过程中不同情况来确定。另外对于Servlet中的属性的使用也会对线程安全产生影响，见下面。

## 自定义的Servlet
通常我们开发自己的Servlet都是继承HttpServlet，然后重写相关方法，这时候线程安全和不安全都是靠我们自己来决定了，没有实例变量的时候，就是线程安全的；而有实例变量的时候，并且会被改变，这时候就不是线程安全的了，需要使用其他手段保证线程安全。另外对于Servlet中的属性的使用也会对线程安全产生影响，见下面。

# 如何控制Servlet的线程安全性
## 变量的线程安全
我们知道当没有实例变量的时候，就基本不存在线程不安全的问题了，所以不使用实例变量是一种方法。
## 属性的线程安全

- ServletContext，不是线程安全的，多线程可以同时读写。使用时要注意。
- HttpSession，不是线程安全的，比如用户打开多个浏览器窗口时候就会产生多个请求针对同一个session的操作。使用时要注意。
- ServletRequest，线程安全的，它对应着一个request请求，所以说是线程安全的。

## SingleThreadModel
我们还可以使用这个接口来创建自己的实现，可以保证线程安全，同一时刻只有一个线程可以执行Servlet实例的service方法，这就成了单线程了，该方式已经被废弃。

# 常用框架的线程安全性
SpringMVC，我们知道Spring的IOC容器默认管理的bean是单实例的，对于SpringMVC的Controller来说也是单实例的，所以开发的时候需要保证线程安全。

Struts1中的action也是单实例的，使用的时候会有线程安全问题。

Struts2中Action会为每一个请求产生一个实例，所以不存在线程安全问题。

注意：当使用Spring管理Struts2的Action时，需要将Action的scope设置为prototype，因为Spring IOC容器中bean默认是单例的。

# DispatcherServlet的线程安全性
在应用启动的时候，就会根据web.xml中配置的有关Spring和SpringMVC的配置启动初始化，对于SpringMVC初始化的是DispatcherServlet，对于Servlet初始化只会进行一次，并且只有一个实例，所以DispatcherServlet只会存在一个。

但是当多线程同时访问DispatcherServlet的时候是线程安全的，因为DispatcherServlet中的内部属性都不会影响线程安全，所以DispatcherServlet可以忽略线程安全的问题。

虽然DispatcherServlet可以认为是线程安全的，但是SpringMVC中的Controller不是。Controller也是单例的，每个请求对应一个Controller中的方法，方法如果没有使用实例变量，可以认为是线程安全的，但是如果有实例变量就要考虑线程安全的问题了。