---
title: Spring源码解析系列之IoC容器初始化
date: 2018-08-06 21:18:46
tags: Spring
---
# IoC容器初始化 #

## 1. IoC容器系列设计与实现 BeanFactory和ApplicationContext##
**在SpringIoC容器设计中，主要由两个容器系列：实现BeanFactory接口的简单容器，另一个是ApplicationContext应用上下文。**
	
- Spring通过定义BeanDefinition来管理基于Spring应用中的各种对象以及它们之间的依赖关系。
- BeanDefinition抽象了Bean的定义，是容器的主要数据类型。
- IoC容器就是用来管理对象依赖关系的，BeanDefinition就是对依赖反转模式中管理的对象依赖关系的数据抽象。

## 2.SpringIoC容器的设计##
##第一条线
- BeanFactory定义了基本IoC容器规范，
	- 提供getBean()基本方法；
- HierarchicalBeanFactory继承了双亲的基本接口，
	- 增加了getParentBeanFactory()接口的功能，使BeanFactory具备了双亲IoC容器的管理功能。
- ConfigurableBeanFactory接口中，主要定义了一些对BeanFactory的配置功能，
	- setParentBeanFactory()设置双亲IoC容器，
	- addBeanPostProcessor()配置Bean后置处理器。
##第二条线
- ApplicationContext应用上下文为核心，主要有
- BeanFactory
- ListableBeanFactory  
	- getBeanDefinitionNames()
- ApplicationContext 
	- 继承MessageSource,ResourceLoader,ApplicationEventPublisher接口，增加了很多高级特性。
- WebApplicationContext || ConfigurableApplicationContext 
##BeanFactory的使用场景
- DefaultListableBeanfactory XmlBeanFactory ApplicationContext都是附加了某种功能的具体实现。
- 通过 & 转义符来得到FactoryBean本身，如&myJndiObject得到FactoryBean
- BeanFactory和FactoryBean区别
	- BeanFactory:就是一个Factory,是IoC容器，或者对象工厂
	- FactoryBean:能够生产或者装饰对象生成的工厂
- XmlBeanFactoryReader对象来读取resource,并回调给BeanFactory;
- Spring使用Resource来封装I/O操作的类，将Resource作为参数传递给BeanFactory的构造函数。IoC容器从而能够定位到BeanDefinition来完成初始化和依赖注入。
##IoC容器使用步骤
1. 创建一个IoC配置文件的抽象资源，资源中包含BeanDefinition信息；
2. 创建一个BeanFactory;
3. 创建一个载入BeanDefinition的读取器，通过回调配置给BeanFactory;
4. 从定义好的资源读取配置信息。完成载入和注册后，IoC容器就初始化起来了。
##ApplicationContext的应用场景
- 支持不同的信息源，多语言开发提供基础；
- 访问资源；ResourceLoader,Resource;
- 支持应用事件；
- 提供附加功能。
##ApplicationContext容器设计原理
- refresh()
- 怎样从文件系统中加载XML的bean定义资源