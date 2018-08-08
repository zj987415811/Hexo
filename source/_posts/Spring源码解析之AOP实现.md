---
title: SpringAOP实现
date: 2018-08-08 14:50:15
tags: SpringAOP
---

## 1.SpringAOP概述 ##
- 常见AOP技术:事务管理，安全检查，缓存，对象池管管理。
	1. 静态代理：AspectJ，使用AOP框架提供的命令进行编译，从而在编译阶段生成AOP代理类，编译时增强；在编译时自动得到一个新类，这个类具有增强原来类的功能。性能更优。
	2. 动态代理：SpringAOP，在运行时借助于JDK动态代理，Cglib等技术在内存中临时生成AOP动态代理类，运行时增强。不需要在编译时增强，而是运行时生成目标类的代理类，该代理类要么与目标接口是相同的接口，要么是目标类的子类。采用运行时动态地，在内存中临时生成代理类。
	3. AOP代理是由AOP框架生成了一个代理对象，该对象作为目标对象使用。AOP代理包含了对目标对象的全部方法，差别在于对于特定切入点添加了增强处理，并回调了目标对象的方法。
	4. Cglib生成代理类:
		1. Enhancer en = new Enhance();
		2. en.setsuperClass(Chinese.class);
		3. en.setCallback(new AroundAvice());
		4. return (Chinese) en.create();
	5. Advice PointCut Advisor(Adivce PointCut)

## 2.Spring AOP的设计与实现 ##
- 动态代理特性:为任意java对象创建代理对象，通过ReflectionAPI来实现。
- InvocationHandler->invoke(Object proxy,Method method, Object[] args)
	- 第一个参数是代理对象
	- 第二个参数是Method方法对象，代表被调用的方法
	- 第三个参数是被调用方法中的参数



- 应用场景：日志和事务
- 建立AopProxy代理对象
	- 设计原理：代理对象的生成，通过配置和调用ProxyFactoryBean来完成。ProxyFactoryBean封装了代理对象的生成，包括Jdk动态代理和Cglib两种方式。

- 配置ProxyFactoryBean
	- 定义通知器Advisor(Advice,PointCut)来定义一个Bean定义了需要对目标对象进行增强的切面行为，也就是Advice通知。
	- 定义ProxyFactoryBean，封装了AOP功能的主要类。需要设定与Aop实现相关的重要属性，如proxyInterface,interceptorNames(通知器名)和target等。
- ProxyFactoryBean生成AopProxy代理对象
	- 在ProxyFactoryBean中，通过interceptorNames属性来配置已经定义好的通知器Advisor；
	- 在ProxyFactoryBean中，需要为target目标对象生成Proxy代理对象，为AOP横切面的编织做好准备工作。
	- ProxyBeanFactory的AOP实现需要依赖JDK或者CGLIB提供的Proxy特性。从FactoryBean中已getObject()方法作为入口完成。
	- getObject()方法是FactoryBean实现的接口，对于target目标对象的增强处理，通过getObject()方法来封装，这些增强是AOP功能的实现提供服务的。
	- getObject()方法首先会对通知器链进行初始化，通知器链封装一系列拦截器，所有拦截器都从配置中读取，为代理对象生成做准备。
	- 生成代理对象时，需要对singleton和prototype进行区分。
	- ProxyFactoryBean#initializeAdvisorChain(){第一次通过ProxyFactoryBean去获取对象的时候，完成初始化后，会读取配置中出现的所有通知器，将通知器的名字交给容器的getBean方法}->getSingletonInstance(){根据AOP框架来判断需要代理的接口}->getProxy()->createAopProxy(){返回AopProxy代理对象，分别为JdkDynamicProxy和Cglib2AopProxy}->isInterface(){jdk动态代理}->createCglibProxy(){非接口}。
