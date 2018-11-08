---
title: docker系列之doker核心原理解读
date: 2018-10-10 21:47:14
tags: docker
---
docker核心原理
<!-- more -->

## 1. docker背后的内核知识 ##
1. docker容器本质上是宿主机上的**进程**。
	1. 通过namespaces实现了资源隔离。
	2. 通过cgroups实现了资源限制。
	3. 通过写时复制实现高效文件操作。
2. 通过namespace资源隔离
	1. chroot---文件系统被隔离
	2. 在分布式环境下进行通信和定位，容器使用独立的IP，端口，路由等。---网络的隔离
	3. 容器需要一个独立的主机名，以便在网络中识别自己。---进程间通信也需要隔离---开发者权限---用户权限
	4. 容器中的应用需要有进程号，需要与宿主机的PID进行隔离。
	5. 6项隔离：
		- 主机名与域名
		- 信号量，消息队列，内存共享
		- 进程编号
		- 网络设备，网络栈，端口
		- 挂载点
		- 用户用户组
	6. 同一个namespace下的进程可以相互感知彼此的变化
3. namespaceAPI的4种方式
	1. 通过clone()在创建新进程的同时创建namespace，是Linux系统调用fork()的一种实现方式，通过flags来控制。
	2. 查看/proc/[pid]/ns文件：如果两个进程指向的namespace编号相同，就说明他们在同一个namespace下，/proc/[pid]/ns里面设置这些link，一旦link文件被打开，只要打开文件描述符存在，那么/uts。
	3. setnx()加入一个已经存在的namespace
	4. 通过unshare()在原先进程上进行namespace隔离
	5. fork()系统调用
		- 程序调用fork()函数时，系统会创建新的进程，为其分配资源，将原来进程的所有值都复制到新进程种，只有少数数值与原来进程值不同，相当于复制了本身。
4.6种namespace
	1. UTS namespace 主机名和域名隔离
	2. IPC namespace 进程间通信:信号量，消息队列，共享内存。申请IPC资源申请一个全局唯一的32位ID,包含了系统IPC标识符以及实现POSIX消息队列的文件系。同一个IPC namespace下进程彼此可见，不同IPC namespace下进程相互不可见。
	3. PID namespace PID重新编号
		1. PID namespace种的init进程
		2. 信号与init进程
		3. 挂载proc文件系统
		4. unshare()和setns()
	4. mount namespace 通过隔离文件系统挂载点对隔离文件系统提供支持。共享挂载
	5. network namespace 
		1. 网络设备
		2. IPv4
		3. IPv6
		4. IP路由器
		5. 防火墙
		6. /proc/net目录
		7. /sys/class/net目录
		8. 套接字
	6. user namespace:隔离了安全相关的标识符和属性，包括用户ID,用户组ID,root目录,key以及特殊权限。
		1. 一个物理的网络设备最多存在于一个network namespace中，可以通过创建veth pair(虚拟网络设备对：有两端，类似于管道，如果数据从一段传入另一端也能接收到，反之亦然) 
		2. 在不同的network namespace间创建通道
		3. 一般情况下，物理网络设备都分配在最初的root namespace中。如果有多块物理网卡，可以把其中一块或者多块分配给新创建network namespace。
		4. 当新创建的network namespace被释放时，就会返回root namespace。而把网络独立出来，给外部用户一种透明的感觉，彷佛在与一个独立网络实体进行通信。
		5. 为了达到该目的，容器的经典的做法就是创建一个veth pair,一端放置在新的namespace中，命名位eth0,一端放在原先的namespace中连接物理网络设备，再通过把多个设备接入网桥或者进行路由转发，来实现通信的目的。
		6. 在建立起veth pair之前，通过管道与新旧namespace进行通信。在docker deamon启动容器的，假设容器初始化的进程称为init。Docker deamon在主机上负责创建这个veth pair,把一端绑定到docker0网桥上，另一端接入新建的network space进程中。这个过程执行期间，Docker deamon和init就通过pipe进行通信。具体来说，Docker deamon完成veth pair的创建之前，init的管道的另一端循环等待，直到管道另一端传来docker deamon关于veth设备的信息，并关闭管道。init才结束等待的过程，并把它eth0启动起来。
	7.  user namespaces

5. cgroup资源限制
	1. 限制资源，为资源设置权重，计算使用量，操控任务启停。
	2. cgroups是linux内核提供的一种机制，这种机制可以根据需求把一些列系统任务以及子任务整合到按资源划分等级的不同组内，从而为系统资源管理提供一个统一框架。
	3. cgroups可以限制，记录任务组所使用的物理资源（CPU,memory,IO），为容器实现虚拟化提供了基本保证，是构建Docker等一些列虚拟化管理工具的基石。
	4. cgroups的4个特点：
		1. cgroups的API以一个伪文件系统的方式实现，用户态的程序可以通过文件操作实现cgroups的组织管理；
		2. cgroups的组织管理操作单元可以细粒度到线程级别，另外用户可以创建和销毁cgroups，从而实现资源再分配和管理；
		3. 所有资源管理的功能都以子系统的方式实现，接口统一；
		4. 子任务创建之初与父任务处于同一个cgroups的控制组。
	5. 本质上来说，cgroups是内核附加在程序上的一系列钩子，通过程序运行时对资源的调度触发相应的钩子以达到资源追踪和限制的目的。
	6. cgroups的作用
		1. 目的是：为不同用户层面的资源管理，提供一个统一化接口。从单个任务的资源控制到操作系统层面的虚拟化，cgroups提供四大功能：
		2. 资源限制：对任务使用的资源总额进行限制，设定应用运行时使用内存上限，超过额度就发出OOM；
		3. 优先级分配：通过分配CPU时间片数量及磁盘IO带宽大小，实际是控制任务的优先级；
		4. 资源统计：统计系统的资源使用量，如CPU使用时长，内存用量等；
		5. 任务控制：对任务执行挂起，恢复等操作。
	7. cgroups术语表
		1. task
		2. cgroup
		3. subsystem
		4. hierarchy
	8. 组织结构与基本规则
		1. 传统任务管理，实际上是先启动init任务作为根节点，再由init节点创建子任务作为子节点，而子节点又可以创建新的子节点。
6. docker架构概览
	1. 使用传统的client-server架构模式，用户通过docker client与docker daemon建立通信，并将请求发送给后者。
	2. APIServer用于接收来自docker client的请求后，根据不同的请求分发给docker daemon的不同模块执行相应的工作。
	3. docker通过driver模块来实现对docker容器执行环境的定制。
	4. 当需要创建docker容器，可以从docker registry中下载镜像，并通过镜像管理graphdriver将下载的镜像以graph形式存储在本地；
	5. 当需要为Docker容器创建网络环境时，则通过网络管理驱动networkdriver创建并配置execdriver来完成。
	6. libcontainer是一个独立的容器管理包，networkdriver和execdriver都通过libcontainer来实现对容器的具体操作，利用uts,ipc,pid,network,mount,user等namespace实现容器之间的资源隔离和利用cgroup实现对容器的资源限制。
	7. 当运行容器的命令执行完毕后，该容器就具有独立的文件系统，安全且相互隔离的运行环境。
7. 模块介绍
	1. docker daemon 
		1. 响应来自docker client的请求，然后将这些请求翻译成系统调用完成容器管理操作；
		2. 该进程会在后台启动一个API server,负责接收Docker client发送的请求；
		3. 接收到的请求将通过docker daemon内部的一个路由分发调度，再由具体的函数来执行请求。
	2. docker client 
		1. 用来向指定的docker daemon发起请求，执行相应的容器管理操作。
		2. 可以是docker命令行工具，也可以是任何遵循docker api的客户端。
		3. graph组件负责维护已下载的镜像信息以及他们之间的关系，大部分docker相关的操作会有graph组件来完成。
		4. graph通过镜像层和每层的元数据来记录这些镜像的信息，用户发起的镜像管理操作最终都转换成了graph对这些层和元数据的操作。
	3. graphdb
		1. 通过graphdb记录它所维护的所有容器节点以及他们之间的link关系，用一个图结构来及逆行保存。
		2. Graph是一个基于SQLite的最简单版本的图形数据库，能够为调用者提供节点增删遍历连接所有父子节点的查询；
		3. 这些节点对应的就是一个容器，而节点间的边就是docker link关系。
		4. 每创建一个容器，就会再graphdb里添加一个几点，而当我们为某个容器设置了link操作后，就会创建一个父子关系，即一条边。
	4. driver
		1. docker daemon负责将用户请求转化成系统调用，进而创建和管理容器的核心进程。为了将这些系统调用抽象成统一的操作接口方便调用者使用，docker把这些操作分类成容器管理驱动，网络管理驱动，文件存储驱动3种。分别对应execdriver,networkdriver和graphdriver。
		2. execdriver是对Linux操作系统的namespace,cgroups,apparmor,SELinux等容器运行所需的系统操作进行一层二次封装，其本质作用类似于LXC,功能更全面。
		3. networkDriver是对容器网络环境进行封装，对于容器来说，网络设备比较独立，并且允许用户进行更多配置。分配通信所需的ip,服务访问的端口和容器与宿主机之间的端口映射，设置hosts,resolv.conf,iptables等。
		4. graphdriver是所有与容器镜像相关操作的最终执行者。graphdriver会在docker工作目录下维护一组与镜像层对应的目录，并记下容器和镜像之间关系等元数据。用户对镜像的操作最终会被映射成这些目录文件以及元数据的增删查改，从而屏蔽掉不同文件存储实现对于上层调用者的影响。 
	5. client和daemon
	6. libcontainer
8. docker镜像管理
	1. bootfs:底层引导文件系统，包括bootloader和操作系统内核，类似于linux引导文件系统。
	2. rootfs:包含了一个操作系统运行所需的文件系统，如目录系统，配置，工具。
		1. 传统的Linux操作系统内核启动时，首先时挂载一个只读的rootfs,当系统检测其完整性之后，再将其切换为读写模式。
		2. 在docker daemon为docker容器挂载rootfs时，将其设为只读模式。
		3. 在挂载完毕后，利用联合挂载技术在已有只读rootfs上再挂载一个读写层。
		4. 可读写层处于docker容器文件系统的最顶层，其下可能联合挂载多个只读层，只有docker容器运行过程中文件系统发生变化时，才会把变化的文件内容写到可读写层，并隐藏只读层种的老版本文件，这技术称为写时复制。
		5. 联合挂载技术:在一个挂载点同时挂载多个文件系统，将挂载点的原目录与被挂载内容进行整合，使得最终可见的文件系统将会包含整合之后的各层文件和目录。
		6. 实现这种联合挂载技术的文件系统同城称为联合文件系统（UNFS）;
		7. 当需要修改镜像内的某个文件时，只对处于文件上方的读写层进行变动，不覆盖下层已有文件系统的内容，已有文件在只读层中的原始版本依然存在，但会被读写层中的新版本文件所隐藏，当docker commit这个修改过的容器文件系统为一个新的镜像时，保存内容仅为最上层读写文件系统中被更新过的文件。
	3. docker镜像相关的概念
		1. registry
		2. repository
		3. index
		4. graph
		5. dockerfile			
9. Docker存储驱动
	1. 存储驱动根据操作系统底层的支持提供了针对某种文件系统的初始化操作以及对镜像层的增删改查和差异比较等操作。
	2. 映射设备，映射表和目标设备。
10. docker数据卷
	1. 读时复制的缺点    
		1. 容器中的文件在宿主机上存在形式复杂，不能在宿主机上很方便地对容器中的文件进行访问。
		2. 多个容器之间数据无法共享。
		3. 当删除容器时，容器产生的数据将丢失。
	2. 引入数据卷机制。
		1. 存在于一个或多个容器中的特定文件或文件夹，这个目录能够独立于联合文件系统的形式在宿主机中存在，并为数据的共享与持久化提供便利。
		2. volume在创建时就会初始化，在容器运行时可以使用其中的文件。
		3. 在不同的容器之间共享和重用。
		4. 数据的操作不会影镜像本身。
		5. 生存周期独立于容器生存周期，即使删除容器，v仍会存在。  
	3. 数据卷原理
		1. 本质是容器中的一个特殊目录。在容器的创建过程中，这个挂载点会被挂载到一个宿主机上的指定目录，这个挂载彷佛是绑定挂载。 
		2. 创建volume:不论用户使用何种方式创建或运行一个待volume的容器，volume的来源只有3种，从容器挂载，从宿主机挂载和从其他容器共享。docker daemon首先需要根据用户指定的volume类型，判断并新建出对应的mount对象。daemon针对mount对象执行如下操作：
			1. 解析参数并生存Mount列表，每一个Mount描述一个volume和容器的对应关系或是一个容器与其它容器共享volume的情况。
			2. 初始化所有的Mount,这一过程在创建容器时执行，即在宿主机和容器文件目录下创建上述Mount对象种所需的路径。
			3. 将Mount对象传递给libcontainer，按照Mount对象中指定的路由，mount参数，读写标志执行所有的mount操作，完成从宿主机到容器内挂载点的映射
11. docker网络管理
	1. 服务虚拟化，存储虚拟化，网络虚拟化
	2. docker网络基础
		1. docker0网桥
			1. 这台多的网卡会在内核路由表上添加一条达到相应网络的静态路由。eth0是容器与外界通信的网卡。
