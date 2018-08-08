---
title: Spring源码系列之SpringMVC容器加载初始化过程解析
date: 2018-08-07 22:55:37
tags: SpringMVC
---
##SpringMVC在Web环境中的加载##
###1.概述###
	SpringIoC是一个独立的模块，并不是直接在Web容器中发挥作用，如果需要在Web容器环境建立使用IoC容器,需要为SpringIoC设计一个启动过程，把IoC容器导入，并在Web容器中建立。在这个过程中，一方面是web容器的启动，另一方面是通过设计特定的web容器拦截器，将IoC容器以拦截的形式载入到web容器中，并将其初始化。而SpringMVC是在IoC容器的基础上建立的MVC运行机制，从而响应Web容器传递的Http请求。

 Tomcat容器中web.xml部署
-
	<servlet>
		<servlet-name />
		<servlet-class />
		<load-on-startup />
	</servlet>
	<servlet-mapping>
		<servlet-name></servlet-name>
		<url-pattern>/*</url-pattern>
	</servlet-mapping>
	<context-param>
		<param-name>contextConfigLocation</param-name>
		<param-value>/WEB-INF/applicationContext.xml</param-value>	
	</context-param>
	<listener>
		<listener-class>
			ContextLoaderListener
		</listener-class>	
	</listener>

- 这里部署了SpringMVC与Tomcat的接口。首先定义了一个Servlet对象；
- 同时部署了了这个Servlet对应的URL映射，这些URL映射为这个Servlet指定了需要处理的Http请求
- context-param参数的配置用来指定Spring IoC容器读取Bean定义的xml文件。在这个xml文件中读取到Spring的配置。
- ContextLoaderListener作为SpringMVC的启动类，被定义为一个监听器，这个监听器与Web服务器的生命周期相关，由ContextLoaderListener监听器负责完成IoC容器在web环境中的启动工作。
- ContextLoaderListener和DispatcherServlet提供了在Web容器中对的Spring,这些接口与web容器耦合是通过ServletContext实现。
- 这个ServletContext为Spring的IoC容器提供了一个宿主环境，在宿主环境中，SpringMVC建立起一个IoC容器的体系。
- 这个体系通过ContextLoaderListener的初始化建立，建立IoC体系后，把DispatcherServlet作为SpringMVC处理web请求的转发器建立起来，从而完成响应Http请求的准备。

##2.上下文在web容器中的启动##


1. IoC容器启动的基本过程

	- IoC容器的启动就是上下文建立过程，与ServletContext相伴而生，是IoC容器在Web环境中的具体表现之一；
	- ContextLoaderListener启动的上下文是根上下文。在根上下文的基础上还有一个与WebMVC相关的上下文来保持控制器需要的MVC对象，上下文的体系是由ContextLoader来完成。
	- contextInitialized()->ContextLoaderListener#initWebApplicationContext()->ContextLoader#loadParentContext()->XmlWebApplicationContext#refresh();
	- 在ContextLoader中，完成了两个IoC容器的建立，一个是Web容器中建立起来的双亲IoC容器，另一个是生成响应的WebApplicationContext并将其初始化。
2. Web容器中的上下文设计---WebApplicationContext
3. ContextLoader设计与实现
	- Spring承载的web应用而言，可以在Web应用程序启动时载入IoC容器(WebApplicationContext)，这个功能由ContextLoaderListener来实现，是在web.xml中配置的监听器。这个监听器通过ContextLoader来完成实际的IoC容器初始化工作。
	- ContextLoader是Spring应用程序在web容器中的启动器。
	- 这个监听器是启动根IoC容器并载入到web容器的主要功能模块，也是整个SpringWeb应用加载IoC的第一个地方。
	- 首先从Servlet事件中得到ServletContext,然后读取配置在web.xml中的相关属性，接着实例化WebApplicationContext,完成其初始化过程。
	- 这个被初始化的第一个上下文是根上下文，这个根上下文载入后，被绑定到web应用的ServletContex上。任何需要访问根上下文的应用程序代码都可以从WebApplicationContextUtils的静态方法中得到。
	- 在ContextLoaderListener中，实现的是ServletContextListener接口，这个接口的方法回结合Web容器的生命周期被调用。ServletContextListener是ServletContext的监听者。如果ServletContext发生了变化，会触发相应的事件，做出预先设计的响应。