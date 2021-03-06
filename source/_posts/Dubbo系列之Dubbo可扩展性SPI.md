---
title: Dubbo系列之Dubbo可扩展性SPI
date: 2018-09-03 10:07:36
tags: Dubbo SPI
---
## 1. Dubbo扩展机制 ##
- 任何系统都是从小系统不断完善的，应当关注当下的需求，然后不断对系统进行迭代。
- 代码层面，需要适当的对关注点进行抽象和隔离，在不断添加和特性时，仍然能保持良好的结构和可维护性，同时允许第三方开发者对其功能及进程扩展。
- 对扩展性的追求甚至超过了性能。
- 良好的扩展性
	- 作为框架的维护这，在添加新功能的时候，只需要添加一些新的代码，而不用大量修改现有代码，符合开闭原则
	- 作为框架的使用者，在添加一个新功能时，不需要去修改框架源码，在自己工程中添加代码即可。
## 2. 可扩展的集中解决方案 ##
- 通常有：
	- Factory模式
	- IoC容器
	- OSGI容器
- Dubbo参考了Java原生的SPI机制，并进行一定扩展，以满足Dubbo需求。
## 3. Java SPI机制 ##
1. Java SPI(Service Provider Interface)是JDK一种动态加载扩展点的实现。
2. 在ClassPath的META-INF/services目录下放置一个接口同名的文本文件，内容为接口的实现类，多个实现类用换行符分割。
3. 缺点：
 	- 需要遍历所有的实现，并实例化，然后才能在循环中找到我们需要的实现。
 	- 配置文件中只是需要简单的列出所有扩展实现，没有命名，很难定位。
 	- 扩展如果依赖其他扩展就做不到自动注入和装配。
 	- 不提供类似于Spring IOC和AOP功能。
 	- 很难和其他框架集成。
## 4. Dubbo扩展点机制 ##
1. 扩展点---Java接口
2. 扩展---扩展点的实现类
3. 扩展实例---扩展点实现类的实例
4. 扩展自适应实例---扩展代理类