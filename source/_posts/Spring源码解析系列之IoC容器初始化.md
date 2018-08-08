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
##IoC容器的初始化过程
- 主要线索：
	- Resource定位:统一的Resource接口来完成，对各种形式的BeanDefinition提供了统一接口。如FileSystemResource,ClassPathResource.
	- 载入:Bean表示为IoC容器内部数据结构->BeanDefinition.
	- 注册:向IoC容器BeanFactory注册BeanDefinition，调用BeanDefinitionRegistry接口实现。内部为将BeanDefinition注入到HashMap中，通过HashMap来持有这些BeanDefinition.
- Resource定位
	- 由refresh()来触发，在FileSystemBeanFactory的构造函数中启动的->createBeanFactory()创建了IoC容器来供ApplicationContext使用；
	- refreshBeanFactory()->getResourceByPath(String path);
	- ClassPathResource resource = new ClassPathResource("bean.xml");
- BeanDefinition的载入和解析
	- 初始化入口:refresh()->初始化BeanDefinition,提供Bean的生命周期管理
	- refresh()->在子类中启动refreshBeanFactory,在此处创建BeanFactory,如果已经有容器存在,就需要把已有的容器销毁或关闭，保证refresh后创建的容器是新的。->prepareBeanFactory->postProcessBeanFactory(beanFactory)->invokeBeanFactoryProcessors(beanFactory)->registerBeanPostProcessors(beanFactory)->initMessageSource()->initApplicationEventMulticaster()->onRefresh()->registerListeners()->finishBeanFactoryInitialization(beanFactory)->finishRefresh();
	- refreshBeanFactory->loadBeanDefinitions(beanFactory)
	- loadBeanDefinitions(beanFactory)中初始化读取器XmlBeanDefinitionReader,并将其在IoC容器中设置好->loadBeanDefinitions(reader)载入过程委托给BeanDefinitionReader来完成->loadBeanDefinitions(Resource resource)在读取器中，得到代表XML文件的Resource,这个对象封装了Xml文件的I/O操作，读取器可以打开I/O流后得到XML的文件对象。通过这个文件对象就可以按照Spring的Bean定义规则来对这个Xml的文档树进行解析了，这个解析过程交个BeanDefinitionDelegate来完成。
	- loadBeanDefinitions(Resource resource)->doLoadBeanDefinitions(inputSource,resource)->Document doc = this,documentLoader.loadocument()获取xml文件的Document对象，解析由documentLoader完成。
	- Spring的Bean语义要求进行解析转化为BeanDefinition数据结构是在registerBeanDefinitions(doc,resource)中完成,具体过程由BeanDefinitionDocmentReader完成，并且对Bean的数量进行统计。
	- BeanDefinition的载入分成两部分:
		- **调用Xml解析器得到document对象**但是并没有按照spring规则进行解析；
		- **按照spring规则进行解析是在documentReader中实现。完成BeanDefinition的处理，结果由BeanDefinitionHolder对象持有**
		- **BeanDefinitionHolder对象除了持有BeanDefinition对象外,还持有其他与BeanDefinition的使用相关的信息，如Bean的名字，别名集合等。**
		- 解析过程由BeanDefinitionParserDelegate来实现。
	- BeanDefinitionParserDelegate完成BeanDefinition的解析，这个类中包含了各种Spring Bean的定义规则的处理。如Bean元素的处理。如id,name,aliase等属性元素。将这些值读出来，设置到BeanDefitionHolder中。
	- 对于其他元素配置的解析，由parseBeanDefinitionElement来完成。解析完成后放入BeanDefinitionHolder中。