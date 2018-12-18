---
title: Docker系列之入门介绍
date: 2018-09-04 16:09:26
tags: Docker

循序渐进学docker笔记
<!-- more -->

## 1. 基本概念 ##
	1. 镜像
	2. 容器
	3. 仓库 Docker Registry
		1. 集中，分发镜像的服务。
		2. 一个Docker Registry中可以包含多个Repository,每个仓库可以包含多个标签，每个标签对应一个镜像。 
		3. 一般而言，一个仓库包含同一个软件不同版本的镜像，标签对应不同的版本。通过<仓库名>：<标签>来指定具体是哪个版本的镜像。没有标签就以latest为默认标签。
	4. 安装 
		1. 阿里云脚本安装:
			curl -sSL http://acs-public-mirror.oss-cn-hangzhou.aliyuncs.com/docker-engine/internet| sh - 
		2. 手动安装
			1. 添加内核参数：
				1. sudo tee -a /etc/sysctl.conf <<-EOF
				   net.bridge.bridge-nf-call-ip6tables = 1
				   net.bridge.bridge-nf-call-iptables = 1
				   EOF
				2. 重新加载sysctl.conf sudo sysctl -p
			 2. 添加yum源：
				 ```sudo tee /etc/yum.repos.d/docker.repo <<-'EOF'
					[dockerrepo]
					name=Docker Repository
					baseurl=https://yum.dockerproject.org/repo/main/centos/7/
					enabled=1
					gpgcheck=1
					gpgkey=https://yum.dockerproject.org/gpg
					EOF```
		     3. 安装docker
			     1. yum install docker-engine
			 3. 建立docker用户组
				 1. 建立docker组：sudo groupadd docker
				 2. 将当前用户加入docker组：sudo usermod -aG docker $USER
				 3. 
## 2.Docker镜像 ##
	1. 操作系统分为内核和用户空间。对于linux而言内核启动后，会挂载root文件系统为其提供用户空间支持。而Docker镜像，就相当于一个root文件系统。
	2. 分层存储
## 3.Docker容器 ##
	1. 镜像和容器就像的类和实例。
		1. 容器的实质是进程，但与直接在属主执行的进程不同，容器进程运行于属于自己的独立命名空间。容器拥有自己的root文件系统，自己的网络配置，自己的进程空间，甚至自己的用户id空间。容器内的进程是运行在一个隔离环境，使用起来就像是在一个独立的宿主的系统一样。使得容器封装的应用比直接在宿主运行更加安全。
		2. 容器不应该向其存储层写入任何数据，容器存储层要保持无状态化。所有文件写入操作，都应该使用**数据卷**，或者绑定宿主目录，这些位置的读写会跳过容器存储层，直接对宿主发生读写，其性能和稳定性更高。
		3. **数据卷的生命周期和容器一样，容器消亡时数据卷不会消亡。使用数据卷后，容器可以随意删除，重新run,数据不会丢失。**
	2. 拉取镜像:docker pull [地址] [仓库名]
	3. 启动容器:docker run --it --rm [地址+仓库名]
	4. 列出镜像:docker images
## 4.镜像 ##
	1. 联合文件系统
	2. 镜像分层结构
	3. 镜像的写时复制
## 5.镜像的命令 ##
	1. docker pull
	2. docker search
	3. docker images
		1. docker images -a 中间镜像层
		2. docker images -f since= ***;
		3. docker images -f label=***;
	4. docker inspect
	5. docker rmi 
	6. docker save 
	7. docker load
## 6.Docker Hub简介 ##
	1. 镜像仓库:docker login -u -p
	2. 搭建仓库
## 7.管理和使用容器 ##
	1. docker create
	2. docker run 
	3. docker ps
	4. docker daemon
	5. docker start
	6. docker stop
	7. docker pause
	8. docker unpause
	9. docker rm
	10. docker top
	11. docker log



#  #**循序渐进学docker笔记**
## 1. docker的结构与特性  ##
1. docker运行于linux系统，是用户态程序，通过一些接口和内核交互，使用了cgroups，namespaces等特性。
2. 工作流程
	1. Docker Client像Daemon发送启动app1指令；
	2. docker daemon发送请求给docker官方仓库，在仓库搜索app1;
	3. 如果找到app1这个应用，就下载到我们服务器上；
	4. 把启动app1应用是否成功的结果返回给Docker Client;
3. docker分层
	1. 把一个应用分为任意多层，比如操作系统是第一层，依赖的库和第三方软件是第二层，应用的软件包和配置文件是第三层。
4. LXC：与宿主机运行在相同的Linux内核，不需要指令级模拟，性能消耗非常小，是非常轻量级的虚拟化容器。

## 2. Docker的基础知识 ##
1. Dokcer的基本概念和常用操作指令
	1. docker inspect 查看容器信息。
		1. dockers inspect -f 使用指定部分信息
	2. docker logs 查看日志
	3. docker stats 命令实时查看容器所占用的系统资源，如cpu,使用率，内存，网络和磁盘开销。
	4. docker Compose
	5. 配置文件

## 3. docker镜像管理 ##
1. 分层：docker history 
2. 新的分层，容器启动后，所有的写操作都会存储在最上面 的可读写层。docker容器可以通过docker commit命令提交生成新镜像。
3. dockerfile
	1. maintainer:镜像创建者
	2. env：设置环境变量
	3. run: 运行shell命令，如果有多条命令可以用“&&”连接
	4. copy：将编译机本地文件拷贝到镜像文件系统中
	5. expose：指定监听的端口
	6. entrypoint:
4. 项目中的镜像分层
## 4. Docker仓库管理 ##
## 5. Docker网络和存储管理 ##
1. docker网络
	1. 默认使用网桥+NAT的通信模型：当docker容器启动时，会创建一对veth虚拟网络设备附加到网桥docker0,另一个加入容器的网络命名空间，并改名eth0。这样就可以在容器与容器之间通过docker0通信了。
	2. 容器与外部网络之间的通信
		1. 容器访问外部网络：创建如下masquerade规则，所有从容器发出的，目的地为host外部网络的包的ip都修改成host的ip,并由host发出去。
		2. 外部网络访容器：如果容器提供的服务需要暴露给外部网络，docker在启动容器时，就会创建SNAT规则。比如：
			docker run -d -p 80:80 apache
2. 网络配置
	1. 网络配置参数
		1. docker进程网络配置
			1. -b:指定docker网桥设备，默认使用docker0;
			2. -bip:指定网桥设备docker0的IP和掩码，使用标准的CIDR形式。
			3. -dns:配置容器的DNS
		2. 容器网络配置：docker run --net
			1. bridge:默认方式；
			2. none:容器没有网络栈，无法与外部通信；
			3. container:<name|id>:使用其他容器的网络栈，
			4. host:容器会使用Host的网络，没有独立的网络栈。
		3. 配置DNS
3. Docker数据管理
	1. docker中容器一旦删除，容器本身对应的rootfs文件系统就会被删除，容器中的所有数据也将随之删除。但是日志或者其他需要持久化的数据不能随容器删除而删除。需要容器与host之前存在共享数据。---**数据卷，可以持久化数据和容器之间共享数据。**
	2. 数据卷
		1. 数据卷中参数：
			1. -v [host-dir]:[container-dir]:[rw|ro] :通过该参数可以给容器创建数据卷，有三个变量：
				1. host-dir:表示host的目录
				2. container-dir:表示容器内部对应的目录
				3. rw|ro:用于控制卷的读写权限
			2. 创建数据卷,在容器内部创建一个数据卷：docker run -it --rm -v /volumel --name test1 ....
			3. 挂载Host的目录作为数据卷
				1. docker run -it --rm -v /data/volumel:/volumel nginx /bin/bash
				2. 通过这种方式可以在Host和容器之间进行数据交换。
				3. 默认情况具有读写权限，也可以挂载只读：-v /data/volumel:/volumel:ro
			4. 挂载Host的文件作为数据卷
				1. docker run --rm -it -v ~/.bash_history:/root/.bash_history 
		2. 数据卷容器：--volumes-from
			1. 创建和挂载数据卷容器
				1. 将一些相关的容器部署在同一个Host上，并且希望这些容器间可以共享数据。此时可以创建一个命名的数据卷容器，然后供其他容器挂载。
				2. docker run -d -v /dbdata --name dbdata traning/postgres echo Data-only container for postgres
				3. docker run -d --volumes-from dbdata --name db1 training/postgres
				4. docker run -d --volumes-from dbdata --name db2 training/postgres 
			2. 数据卷容器的应用
				1. 日志收集
			3. 备份，恢复和迁移数据卷
				1. 备份数据卷：--volumes-from
					1. docker run --volumes-from dbdata -v ${pwd}:/backup nginx tar cvf /backup/backpu.tar /dbdata
				2. 恢复数据卷
					1. docker run -v /dbdata --name dbdata2 nginx /bin/bash
					2. docker run --volumes-from dbdata2 -v ${pwd}:/backup busybox tar xvf /backup/back.tar
		3. docker存储驱动(storage driver)
			1. docker存储驱动历史：
				1. graph driver--->AUFS--->没有进入内核主线
				2. device-mapper
				3. btrfs
				4. overlayfs 
			2. docker overlayfs driver
				1. 使用overlayfs的lowerdir指向image layer，使用upperdir指向containerlayer,merged将lowerdir于upperdir整合起来提供统一视图给容器，作为根文件系统。
4. docker项目日常维护
	1. 宿主机管理
		1. docker 安装启动：
			1. 安装：yum install docker-in
			2. 启动：service docker start 
		2. 检查docker进程是否启动
			1. ps aux | grep docker 
		3. 转移数据
			1. service docker stop 
			2. mkdir /data/dockerData/
			3. mv /var/lib/docker /data/dockerData/
			4. ln -s /data/dockerData/docker/ /var/lib/docker
			5. service docker start 
		4. 升级到最新版本
	2. 网桥模式：默认时NAT模式，如果配置网桥模式需要三步：
		1. 宿主机配网桥：
			1. 备份原来网卡配置：
			2. 安装pipework脚本；
			3. 更新iproute。
	3. GitLab的日常维护
## 6. Docker Swarm容器集群 ##