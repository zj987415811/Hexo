---
title: k8s系列之入门
date: 2018-09-27 16:02:12
tags: k8s
---
k8s入门讲解
<!-- more -->

# 第一章 Kubernetes入门 #
1. K8S是什么
	1. 大规模集群管理系统，基于容器技术，实现资源管理的自动化，以及跨多个数据中心得资源利用率最大化。
	2. TCP通信协议进行交互。
	3. 完备得分布式系统支撑平台，具有完备得集群管理能力，多层次得安全防护和准入机制，多租户应用支撑能力，透明的服务注册和服务发现机制，内建智能负载均衡器，强大的故障发现和自我修复能力，服务滚动升级和在线扩容能力，可扩展的资源自动调度机制，多粒度的资源配额管理能力，完善的管理工具，如开发，部署测试，运维监控等各个环节。
2. K8S基础
	1. Service是分布式集群架构的核心，一个Service对象拥有如下关键特征：
		1. 拥有一个唯一指定的名字；
		2. 拥有一个虚拟ip和端口号；
		3. 能够提供某种远程服务能力；
		4. 被映射到了提供这种服务能力的一组容器应用上。
	2. Service的服务进程目前都基于Socket通信方式对外提供服务，如redis,Memcache,MySql,WebServer,或者一个特定的TCP server进程。Service通常由多个相关的服务进程来提供服务，每个服务进程都有一个独立的IP+PORT访问点，从来连接到指定的Service上。
	3. 强大的隔离功能，K8S设计了Pod对象，将每个服务进程包装到相应的Pod中，使其成为Pod中运行的一个容器。为了建立Service和Pod间的关联关系，K8S给Pod贴上一个Label，给运行MySQL的Pod贴上name=mysql标签。
	4. Pod在节点中运行，节点可以是物理机，可以是云中的一个虚拟机，每个Pod里面运行着一个特殊的Pause容器，其他容器运行业务容器，这些业务容器工厂Pause容器的网络栈和Volume挂载卷，因而通信和数据交换更为高效。因此密切相关的服务进程放入同一个Pod中。
	5. 集群管理:将集群中机器划分为一个Master节点和一群工作节点，Master上运行着集群管理相关的一组进程kube-apiserver,kube-controller-manager和kube-scheduler,这些进程实现了整个集群资源管理，Pod调度，弹性伸缩，安全控制，系统监控和纠错等管理功能，并且都是全自动完成的。Node作为集群中的工作节点，运行真正的APP,在Node上K8S管理的最小运行单元式Pod。Node上运行着Kubelet,kube-proxy服务进程，这些服务进程负责Pod的创建，启动监控，重启销毁以及软件模式的负载均衡。
3. 使用K8S的原因
	1. IT从来都是技术驱动的。
	2. 简化团队，架构师专注于系统服务组件的提炼，开发专注于业务代码，运维负责K8S的部署和运维。
	3. 全面拥抱微服务，微服务架构的核心是将一个巨大的单体应用分解维很多小的相互连接的微服务，一个微服务背后有多个实例副本在支撑，副本数量随着系统的负荷变化而进行调整，内嵌的负载均衡器在这里发挥重要作用。
	4. 很方便的搬迁到公有云上。
	5. 超强的横向扩容能力。
4.K8S基本概念和术语
	1. Node---相对于master而言的的工作主机。每个node上运行用于启动和管理的Pod服务-Kubelet，并能够被Master管理。在node上运行的是服务进程包括Kubelet,Kube-proxy,docker deamon。
	2. Node的信息:
		1. Node地址：主机Ip地址+NodeID
		2. Node运行状态:Pending,Running,Terminated三种状态
		3. Node Condition(条件):描述Running状态Node的运行条件，Read---健康状态，可以接收从Master发来的创建Pod指令。
		4. Node系统容量:描述Node可用的系统资源，包括CPU,内存数量，最大可调度Pod数量等。
		5. 其他:内核版本，K8S版本，docker版本，操作系统名称。
	3. Pod
		1. 最基本的操作单元，包含一个或多个紧密相关的容器。是被容器化的环境看作应用层的逻辑宿主机，一个Pod中的多个容器应用是紧耦合的。Pod在node上被创建，启动或者销毁。
		2. 在Pod上封装Node的原因:Docker容器之间的通信收到Docker网络机制的限制，只能通过Link方式才能访问另一个容器提供的服务。大量容器之间的Link是一种重量级连接。通过Pod将多个容器组合在一个虚拟的主机内，可以通过Localhost就能通信。
		3. 一个Pod中的应用容器共享同一组资源：
			1. PID命名空间：Pod中的不同应用程序可以看到其他应用程序的进程ID;
			2. 网络命名空间：Pod中的多个容器能够访问同一个IP和端口范围；
			3. IPC命名空间：Pod中的多个容器能够使用SystemV IPC或者POSIX消息队列进行通信；
			4. UTS命名空间：Pod中的多个容器共享一个主机名；
			5. Volumes（共享存储卷）:pod中的各个容器可以访问在Pod级别定义的Volumes。
		4. Pod定义通过Yml或者json格式配置文件来完成。
		5. Pod的生命周期是通过Replication Controller来管理的。Pod的生命周期过程包括:在模板进行定义，然后分到Node上运行，在Pod所含容器运行结束后Pod也结束。在整个过程中，Pod处于一下4种状态之一：
			1. Pending:Pod定义正确，提交Master，但其包含的容器镜像还未完全创建。通常Master对Pod进行调度需要一些时间，之后Node对镜像进行下载也需要一些时间。
			2. Running:Pod已被分配到某个Node上，但其包含的所有容器镜像都已经创建完成，并成功运行起来。
			3. Suceeded:Pod中所有所有容器都成功结束，并且不会重启，这是Pod最终状态。
			4. Failed:Pod中所有容器都结束了，但至少一个容器以失败状态结束的，这是Pod的一种最终状态。
	4. Label(标签)
		1. Label以key/value键值对的形式附加到各种对象上，如Pod,Service,RC,Node等。Label定义了这些对象的可识别属性，用来对它们进行管理和选择。Label可以在创建对象时附加到对象上，也可以在对象创建后通过API进行管理。
		2. 在为对象定义好Label后，其他对象就可以使用Label Selector选择器来定义其作用的对象。
			"Labels":{
				"key1":"value1",
				"key2":"value2"
			}	
		3. Label Selector时，看作SQL查询语句种的where查询条件的语法。
	5.Replication Controller(RC)
		1. 用于定义Pod副本的数量。在Master内，Controller Manager进行通过RC的定义来完成Pod的创建，监控，启停等操作。
	6. Service
		1. Pod都会分配一个单独的IP地址，但是会随着Pod的销毁而消失。如果一组Pod组成一个集群来提供服务，如何访问；
		2. 一个Service可以看作一组提供相同服务的Pod对外访问接口。Service作用于那些Pod是通过Label Selector来定义的。
		3. Pod的IP地址和Service的Cluster的IP地址
			1. Pod的IP地址是Docker Daemon根据docker0网桥的IP地址段进行分配的，但是Service的ClusterIP地址是K8s系统种的虚拟IP地址，由系统动态分配。service的ClusterIP地址相对于Pod的IP地址来说相对稳定，Service被创建时被分配一个IP地址，销毁该Service之前，这个IP地址都不会再变化了。Pod在K8S集群中生命周期较短，ReplicationController销毁，再次创建，会分配一个新的IP地址。
		4. 外部访问Service
			1. Service对象在ClusterIPRange池中分配的IP只能在内部访问，所以其他Pod都可以无障碍地访问到它。如果这个Service作为前端服务，准备为集群外的客户端提供服务，就需要分配公共IP;
			2. K8S支持两种对外提供服务的Service的type定义:NodePort和LoadBalancer
			3. NodePort
			4. LoadBalancer
	7. Volume(存储卷)
		1. Volume是Pod中能够被多个容器访问的共享目录。K8S的Volume概念与Docker的Volume比较类似。
		2. Volume类型
			1. EmptyDir:一个EmptyDir Volume是在Pod分配到Node时创建得。从它的名称就可以看出，初始为空。同一个Pod中所有容器可以读些EmptyDir中得相同文件。当Pod从Node上移除时，EmptyDir中得数据也会永久被删除。
				1. 用途:临时空间，例如用于某些应用程序运行时所需得临时目录，且无须永久保留。长时间任务的中间过程CheckPoint临时保存目录。一个容器需要从另一个容器中获取数据的目录。
				2. tmpfs
			2. hostPath:在Pod上挂载宿主机上的文件或目录。
				1. 容器应用程序生成的日志文件需要永久保存
	8. Namespace
		1. Namespace是k8s系统中另一个非常重要的概念，通过将系统内部的对象分配到不同的Namespace中，形成逻辑上分组的不同项目，小组，用户组，便于不同的分组在共享使用整个集群的同时还能被分别管理。
	9. Annotation
5. K8S总体架构
	1. k8s集群由两类节点组成:Master和Node,Master上运行etcd,API Server,Controller Manager和Scheduler四个组件，后三个组件构成了K8S的总控中心，负责对集群中所有资源进行管控和调度。
	2. 在每个Node上运行Kubelet,Proxy和Docker Daemon三个组件，负责对本节点上的Pod的生命周期进行管理，以实现服务代理的功能。
	3. 所有节点上都可以运行Kubectl命令行工具，提供了K8S的集群管理工具集。
	4. etcd是高可用的key/value存储系统，用于持久化存储集群中所有的资源对象，如Node,Service,Pod,RC,Namespace等。API Server则提供了操作etcd的封装接口API，以Rest方式提供服务，这些API基本上都是集群中资源对象的增删改查及监听资源变化的接口，如创建Pod,创建RC,监听Pod的变化等接口。API Server是连接其他所有服务组件的枢纽。
	5. RC与相关Service创建完整流程
		1. 通过Kubectl提交一个创建RC的请求，该请求通过APIServer被写入etcd;
		2. 此时Controller Manager通过API Server的监听资源变化的接口监听到这个RC事件，分析之后，发现当前集群中没有Pod实例。
		3. 于是根据RC的Pod模板定义生成一个Pod对象，通过API Server写入etcd中，接下来此事件被Scheduler发现，执行一个复杂的调度流程，为这个新的Pod选定一个落户的Node，这个过程称为绑定（Pod Binding）。
		4. 通过API Server将这一结果写入到etcd中，随后目标Node上运行的Kubelet进程通过API Server检测到这个新生的Pod并按照定义，启动改Pod并负责其下半生，直到Pod的生命走到尽头。
		5. 通过Kubetcl提交一个映射到该Pod的Service的创建请求，Controller Manager会通过Label标签查询到相关联的Pod实例，然后生成Service的EndPoints信息并通过API Server写入到etcd中。
		6. 接下来所有Node上运行的Proxy进程通过API Server查询并监听Service对象与其对应的Endpoints信息，建立一个软件方式负载均衡器来实现service访问到后端Pod的流量转发功能。
	6. 组件的功能
		1. API Server:提供了资源对象的唯一操作入口，其他所有组件都必须通过它提供的API来操作资源数据，通过相关的资源数据"全量查询"+"变化监听"，这些组件可以实时地完成相关业务功能，如某个新的Pod一旦被提交到API Server中，C M就会立即发现并开始调度。
		2. Controller Manager:集群内部的管理控制中心，其主要目的是实现K8S集群自动检测和恢复的自动化工作，比如根据RC的定义完成Pod的复制或移除，以确保Pod实例数符合RC副本的定义；根据Service与Pod的管理关系，完成服务的EndPoints对象的创建和更新；其他诸如Node的发现，管理和状态监控，死亡容器所占磁盘空间及本地缓存的镜像文件的清理工作也是由Controller Manager完成的。
		3. Scheduler:集群的调度器，负责Pod在集群节点中的调度分配。
		4. Kubelet:负责本Node节点上的Pod创建，修改，监控，删除等全生命周期管理，同事Kubelet定时上报本节点的状态信息到API Server里。
		5. Proxy:实现了Service的代理及软件模式的负载均衡器。
		6. 客户端通过Kubectl命令行工具或Kubetcl Proxy来访问K8S系统，在K8S集群内部的客户端可以直接使用Kubetcl命令管理集群。Kubectl Proxy是APIServer的一个反向代理，在K8s集群外部的客户端可以通过Kubectl Proxy来访问API Server。
		7. API Server内部有一套完备的安全机制，包括认证，授权，以及准入控制等相关模块，APIServer在收到Rest请求后，会首先执行认证授权和准入控制的相关逻辑，过滤掉非法请求，然后将请求发送给API Server中的REST服务模块去执行资源的具体操作逻辑。
		8. Node节点运行的K8S服务内嵌了一个cAdvisor服务，用于实时监控Docker上运行的容器指标。
