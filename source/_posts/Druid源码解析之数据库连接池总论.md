---
title: Druid源码解析之数据库连接池总论
date: 2018-08-20 18:17:52
tags: Druid 数据库连接池
---
## 1.应用程序直接获取数据连接的缺点 ##
1. 用户每次请求都需要向数据库获得连接，数据库创建连接通常需要消耗大量资源，创建时间也较长。
2. 大量得创建极大浪费数据库得资源，并且极易造成数据库服务器内存溢出，宕机。、

## 2.使用数据库连接池优化程序性能 ##
1. 数据库连接池的概念
	1. 数据库连接是一种关键得有限得昂贵得资源，咋多用户网页程序中体现尤为突出。
	2. 对数据库连接的管理能力显著影响到整个应用程序的伸缩性和健壮性，影响到程序的性能指标。
	3. 数据库连接池负责分配，管理和释放数据库连接，它允许应用程序重复使用一个现有的数据库连接，而不是重新建立一个。
	4. 数据库连接池在初始化时将创建一定数量的数据库连接池放到连接池中，这些数据库连接的数量由最小数据库连接数来设定。无论这些数据库连接是否被使用，连接池都将保证至少拥有这么多的连接数量。连接池的最大数据库连接数量限定了这个连接池能占有的最大连接数，当应用程序向连接池请求连接数超过最大连接数量时，这些请求将会被加入到等待队列中。
	5. 数据库连接池的最小连接数和最大连接数的设置需要考虑一下几个因素：
		1. 最小连接数:是连接池一直保持的数据库连接，所以如果应用程序对数据库连接使用量不大，将会有大量数据库连接资源被浪费；
		2. 最大连接数:是连接池能申请的最大连接数，如果数据库连接请求超过次数，后面的数据库连接请求将被加入到等待队列中，这会影响到以后的数据库操作。
2. 编写数据库连接池
	1. 实现java.sql.DataSource接口,DataSource接口定义了两个重载的getConnection方法:
		1. Connection getConnection();
		2. Connection getConnection(String username, String password);
	2. 实现DataSource接口，并实现连接池功能的步骤：
		1. 在DataSource构造函数中批量创建与数据库的连接，并把创建的连接加入LinkedList对象中；
		2. 实现getConnection方法，让getConnection方法每次调用时，从LinkedList中去一个Connection返回给用户；
		3. 当用户使用完Connection,调用Connection.close()方法时，Collection对象应保证自己返回到LinkedList中，而不是把conn还给数据库。**Collection保证将自己返回到LinkedList中**；
	3. 部分核心代码
		`proxyConn = (Connection) Proxy.newProxyInstance(this.getClass().getClassLoader(),conn.getClass().getInterfaces(),new InvocationHandler(){
			public Object invoke(Object proxy,Method method,Object[] args) {
				if(method.getName().equals("close")) {
					pool.addLast(conn);
					return null;
				}
				return method.invoke(conn,args);
			}
		})` 
3. 开源数据库连接池
	1. 很多WEB服务器都提供了DataSource的实现，即连接池的实现。把DataSource的实现，按其英文含义称之为数据源，数据源中包含了数据库连接池的实现。
	2. DBCP C3P0 使用了数据库连接池，就不需要在编写连接数据库代码了，直接从数据源获得数据库的连接。
	3. DBCP数据源:tomcat采用该连接池
	4. C3P0数据源:dbcp没有自动回收空闲连接的功能，c3p0有自动回收空闲连接功能
	5. Tomcat数据源