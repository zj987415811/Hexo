---
title: Druid源码解析之初步认识
date: 2018-08-10 22:38:12
tags: Druid
---

##Druid源码解析##
- Druid是什么
	- 数据库连接池，提供强大的监控与扩展功能，包含ProxyDriver,一系列内置的JDBC组件库，一个SQL Parser;
	- 监控组件，监控应用程序的运行情况，包括Web URI,Spring,JDBC等。为了监控执行SQL执行情况，做了一个Filter-Chain模式的ProxyDriver,缺省提供给StatFilter。还做了一个SQL Parser，将数据库连接池，SQL Parser,ProxyDriver合起来做了一个项目，名为Druid.
	- 亚秒级查
	- 支持JDBC兼容的数据库，包括Oracle,MySQL,DB。。。等等
	- 提供快速的聚合能力以及亚秒级的OLAP查询能力，多租户的设计，面向用户分析应用的理想方案
	- 阿里巴巴线上验证
	- 强大的监控特性，通过Druid提供的监控功能，可以清楚知道连接池和SQL工作情况,监控SQL执行时间，ResultSet持有时间，返回行数，更新行数，错误次数，错误堆栈信息。
		- SQL执行的耗时区间分布。
		- 监控连接池的物理连接创建和销毁次数，逻辑连接的申请和关闭次数，非空等待次数,PSCache命中率等。
		- 为了方便扩展，提供了Filter-Chain模式的扩展API,可以自己编写Filter拦截JDBC中的任何方法，如性能监控，SQL审计，用户名密码加密，日志等待。
		- 内置StatFilter,日志输出的Log系列Filter,防御SQL注入攻击的WallFilter。
		- ExceptionSorter,当一个连接产生不可恢复的异常时，必须立即从连接池中逐出，否则会产生大量错误。
		- PSCache内存优化对于支持游标的数据库,大幅度提升SQL执行性能。一个PreparedStatement对应服务器一个游标，可以缓存起来重复执行。
		- LRU
- Druid如何扩展JDBC？
	- Druid在DruidDataSource和ProxyDriver上提供了Filter-Chain模式的扩展API,类似Servlet的Filter;