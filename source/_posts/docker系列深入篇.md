---
title: docker系列深入篇
date: 2018-10-08 19:00:51
tags: docker
---
docker deep learning

<!-- more -->

## 1. 数据卷 ##
1. 网络通信和文件读写
2. 数据卷用于存储文件数据的特殊模块
3. 数据卷：一个挂载在容器内文件系统中的文件或目录。
4. 使用docker create或者docker run创建容器时，可以通过-v参数向容器中挂载一个数据卷。
5. 使用docker rm -v /path删除数据卷
## 2. 网络 ##
## 3. 制作镜像 ##
## 4. 网络进阶 ##
1. Network Namespaces
	1. docker的容器技术是基于Linux Kernel提供的Linux Container。在Linux Container中，包含了能够隔离程序进程的Namespaces，在Namespaces技术里，存在一个用于隔离程序对网络信息调用的子模块---Network Namespaces。
	2. 每个被NetWork Namespaces隔离的空间都拥有自己的网络设备，ip地址，录用表，防火墙，端口表等。     
2. Veth Pair
	1. 一个虚拟的网络通道，将通道一段的数据传输到另外一段。实现了容器所隔离的网络环境与容器外部的网络通信。
	2. Veth Pair提供两个端点，只能连接某两个网络终端。
3. Linux Bridge把Veth Pair的一端都连接到宿主机中某个网桥所构成的交换机中，把容器连接不同的外部网络的任务由Veth Pair转移到网桥上。
4. Iptables
	1. 用于管理网络过滤的程序，能够根据指定的规则，对位于Linux内核中的netfilter进行操作，实现对网络信息包的过滤功能。
	2. 利用Iptables可以实现端口的映射，让宿主机外部访问到容器的端口是通过端口映射来实现。
5.Docker网络主要由Network Namespaces,Veth Pair,Linux Bridge,Iptables等技术实现。
	1. Network Namespaces:实现了网络资源隔离，对隔离环境提供了网络设备，协议栈，路由表，防火墙，/proc/net目录，/sys/class/net目录，端口表等网络配置。
	2. Veth Pair:实现了打穿隔离环境的网络传输数据通道。在Docker中，它的一段连接到容器的虚拟网卡上，另一端连接到宿主机专用的网桥上，通过这种方式实现了Docker容器与外部网络的互通。
	3. linux bridge:在置在宿主机中的网桥，起到网络交换机的作用。容器网络通过Veth Pair连接到网桥，能够在容器间转发网络数据。
	4. Iptables:  用于提供网络数据透传，NAT功能，实现docker网络防火墙等网络安全防护的需求。
6.网络模型
	1. CNM Container Network Model
		1. Sandbox 网络沙盒 
		2. Endpoint 端点
		3. NetWork 网络
	2. docker中的网络
		1. 默认网络   
		2. 自定义网络
		3. 容器与外部通信
	3. 网络实践
		1. 管理容器网络
		2. 容器连接网络
		3. 配置docker0网桥
		4. 自定义网桥
		5. 配置DNS
		6. 使用IPV6
## 5.安全 ##    
1. 命名空间隔离
	1. 仅仅让不同的命名空间独立享有进程信息，网络信息等。
2. 资源控制组	
	1. 主要功能是CPU,内存,硬盘IO等程序访问的计算机资源进行控制。
	2. docker中，每个容器都拥有一个独立的控制组策略集，用来控制容器中每个程序对计算机资源的访问和调用。
	3. 通过控制组:
		1. 资源限制
		2. 优先化
		3. 用量报告
3. 内核能力机制     
4. 资源使用限制                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                