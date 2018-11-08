---
title: Netty系列之Netty实战读书笔记
date: 2018-10-11 20:21:43
tags: netty
---

 netty实战的笔记介绍

<!-- more -->

## 第一章 Netty---异步和事件驱动 ##
1. netty是一款异步的事件驱动的网络应用程序框架，支持快速地开发可维护的高性能面向协议的服务器和客户端。
2. 核心组件
	- Channel
	- 回调
	- Future
	- 事件和ChannelHandler  
## 第三章 Netty的组件和设计 ##
### 3.1. Channel,EventLoop,ChannelFuture ###
	1. Channel
		1. 基本的I/O操作（bind(),connect(),read(),write()）依赖于底层网络传输所提供的原语。在java网络编程中，基本构造是class socket。Netty的Channel接口提供的API大大降低了直接使用socket类的复杂性。

	2. EventLoop接口:定义了核心抽象，用于处理连接生命周期中所发生的事件。
		1. 一个EventLoopGroup包含一个或多个EventLoop
		2. 一个EventLoop在它的生命周期内之和一个Thread绑定
		3. 所有由EventLoop处理的IO事件都将在它专有的Thread上被处理
		4. 一个Channel在他的生命周期内只注册于一个EventLoop;
		5. 一个EventLoop可能会被分配给一个或多个Channel。
	3. ChannelFuture
		1. 所有的I/O操作都是异步的，一个操作可能不会立即返回，需要一种用于在之后某个时间点确定其结果的方法。
		2. Netty提供了ChannelFuture接口，其addListener()方法注册一个ChannelFutureListener,以便在某个操作完成时得到通知。
### 3.2. ChannalHandler和ChannelPipeline ###
1. ChannelHandler接口
	1. 充当了所有处理入站和出战数据的应用程序逻辑容器。
	2. ChannelHandler由网络事件触发，用于几乎所有的的事件动作。
	3. Netty以适配器的形式提供了大量默认的ChannelHandler实现，用于简化程序处理逻辑的开发过程。ChannelPipeline中的每个ChannelHandler将负责将事件转发到链中的下一个ChannelHandler。这些适配器就会执行这个操作。
	4. 适配器类：一些适配器类可以将编写自定义ChannelHandler所需要的努力降低，提供了定义在对应接口中所有方法的默认实现。如：ChannelHandlerAdapter,ChannelInboundHandlerAdapter,ChannelOutboundHandlerAdapter,ChannelDuplexHandler.
	4. 
2. ChannelPipeline接口
	1. 为ChannelHandler链提供容器，并定义了用于在该链上传播入站和出战事件流的API。
	2. 当Channel被创建时，它会被自动地分配到它专属的ChannelPipline。
	3. ChannelHandler安装到ChannelPipeline中的过程：
		1. 一个ChannelInitializer的实现被注册到ServerBootstrap中；
		2. 当ChannelInitializer.initChannel()方法被调用时，ChannelInitializer将在ChannelPipeline中安装一组自定义的ChannelHandler;
		3. ChannelInitializer将它自己从ChannelPipeline中移除。
	4. ChannelHandler是专为支持广泛的用途而设计的。可以是来往ChannelPipline事件的任何代码的通用容器。
	5. 使得**事件流经ChannelPipline**是ChannelHandler的工作。在应用程序的初始化或引导阶段被安装。这些对象接收事件，执行他们所实现的处理逻辑，并将数据传递给链中的下一个ChannelHandler。他们的执行顺序由被添加的顺序所决定的。也就是说ChannelPipeline是ChannelHandler的编排器顺序。
	6. Netty中有2种发送消息的方式，可以直接写到Channel中，也可以写到和ChannelHandler相关联的ChannelHandlerContext对象中。前一种方式导致消息从ChannelPipeline的尾端开始流动，而后者导致消息从ChannelPipeline中的下一个ChannelHandler开始流动。
3. 编码器，解码器和ChannelHandler子类SimpleChannelInboundHandler-ChannelInboundHandlerAdapter。
4. 编码器和解码器
5. 抽象类SimpleChannelInboundHandler
	1. 用ChannelHandler来接收解码消息，并对该数据应用业务逻辑。
	2. 只需扩展基类SimpleChannelInboundHandler<T>,T是要处理的消息的Java类型。在这个ChannelHandler中要重写基类的一个或者多个方法，并且获取一个到ChannelHandlerContext的引用，将这个引用作为输入参数传递给ChannelHandler的所有方法。
	3. 在这种类型的ChannelHandler中，最重要的方法是channelRead0(ChannelHandlerContext context,T);
### 3.3 引导 ###
1. Netty的引导类为应用程序的网络配置提供了容器，如将一个进程绑定到某个指定的端口，或者将一个进程连接到另一个运行在某个指定主机的指定端口进程。
2. 客户端和服务器

## 第四章 传输 ##
### 4.1 传输API ###
1. 核心是interface Channel,用于所有的IO操作。
2. 每个Channel都会分配一个ChannelPipeline和ChannelConfig。ChannelConfig包含该Channel所有的配置设置，并且支持热更新。同时也有可能是实现ChannelConfig的子类型。
3. Channel是独一无二，并且保证顺序，需要将其生命为Comparable的子接口。同时返回不同的散列码。如果相同就会抛错。
4. ChannelPipline持有所有将应用于入站和出战数据以及事件的ChannelHandler实例。这些ChannelHandler实现了应用程序用于处理状态变化以及数据处理的逻辑。
5. ChannelHandler典型用途：
	1. 将数据从一种格式转化为另一种格式；
	2. 提供异常通知；
	3. 提供Channel变为活动或者非活动；
	4. 提供Channel注册到EventLoop或者从EventLoop注销时的通知。
	5. 提供用于相关的自定义事件通知。
6. 通过根据需要添加或者移除Channel实例来修改ChannelPipeline。
### 4.2 内置的传输 ###
1. Netty内置了一些可开箱即用的传输。NIO，Epoll,OiO,Local,Embedded
2. NIO
	1. 提供了一个所有I/O操作全异步的实现。利用了自身NIO子系统被引入JDK时候可用的基本选择权的API;选择器背后的概念就是一个注册表，将可以请求在Channel的状态发生变化时得到通知。可能的变化有：
		1. 新的Channel连接已经完成；
		2. Channel连接已经完成；
		3. Channel有已经就绪的可供读取的数据；
		4. Channel可用于写数据。
	2. OP_ACCEPT:请求在接受新连接并创建Channel时获得通知；OP_CONNECT:请求在建立一个连接时获得通知；OP_READ:请求当数据已经就绪，可以从Channel中读取时获得通知。OP_WRITE:请求当可以向Channel中写更多数据时获得通知，这处理了套接字缓冲区被完全填满时得情况，这种情况通常发生在数据的发送速度比远程节点可处理的数据更快的时候。
	3. 流程
		1. 新的Channel注册到选择器；
		2. 选择器处理状态变化的通知；
		3. 之前已注册的Channel;
		4. Selector.select()将会阻塞，直到接收到新的状态或者配置超时时间已过；
		5. 检测状态是否变化，处理所有状态变化；
		6. 在选择器运行同一线程中执行其他任务。	
	4. 零拷贝
		1. 在NIO和Epoll传输时使用的特性。可以快速高效地将数据文件从文件系统移动到网络接口，而不需要复制到内核空间，如在FTP或者HTTP中有显著性能提升。
		2. 只能传输原始文件。
3. Epoll
## 第5章 ByteBuf ##
### 5.1 ByteBuf的API ###
1. ByteBuf
	1. 被用户自定义的缓冲区类型扩展；
	2. 通过内置的复合缓冲区类型实现了透明的零拷贝；
	3. 容量可以按需增长；
	4. 在读写之间切换不需要调用ByteBuffer的flip()方法；
	5. 读写使用了不同的索引；
	6. 支持方法的链式调用；
	7. 支持引用计数；
	8. 支持池化。
### 5.2 ByteBuf类---Netty数据容器类  ###
1. 工作原理
	1. ByteBuf维护了两个不同的索引：一个用于读取，一个用于写入。当从ByteBuf读取时，其readerIndex将会被递增已经被读取的字节数。当写入ByteBuf时，writeIndex也会被递增。
	2. read和write开头的ByteBuf方法，将会推进其对应的索引，而名称以set或get开头的操作则不会。后面的这些方法将在作为了一个参数传入的一个相应索引上执行操作。
		3. 可以指定ByteBuf的最大容量，试图移动索引超过这个值将会触发一个异常。
2. ByteBuf的使用模式
	1. 堆缓冲区
		1. 最常用的是ByteBuf模式是将数据存储在JVM的堆空间中。这种模式称为支数组。能够在没有池化的情况下快速的分配和释放。
		2. bytebuf.array();
	2. 直接缓冲区
		1. 在JDK1.4中，引入了ByteBuf允许JVM实现通过本地调用来分配内存。避免了每次调用本地I/O操作之前，将缓冲区的内容复制到中间缓冲区。
		2. 直接缓冲区内容将驻留在常规的会被垃圾回收的堆之外。如果在堆上分配的缓冲区，在通过套接字发送它之前，jvm会在内部将缓冲区复制到直接缓冲区中。
		3. 缺点：
			1. 分配和释放较为昂贵。
	3. 复合缓冲区
		1. 多个ByteBuf提供一个聚合视图。根据需要添加或者删除ByteBuf实例。
		2. ByteBuf通过CompositeByteBuf实现这个模式，提供了一个将多个缓冲区表示为单个合并缓冲区的虚拟表示。
### 5.3 字节级操作  ###
1. 随机访问索引
	1. 与普通java字节数组一样，ByteBuf索引从零开始。
2. 顺序访问索引
	1. ByteBuf同时具有读索引和写索引
3. 可丢弃字节
4. 可读字节
5. 可写字节
6. 索引管理
7. 查找操作
8. 派生缓冲区
9. 读写
### 5.4 ByteBufHolder接口 ###
1. 除了数据负载之外，还需要存储各种属性值。
2. 为了处理这些情况，有了ByteBufHolder。
### 5.5 ByteBuf分配 ###
1. 按需分配:ByteBufAllocator接口
	1. 池化
	2. PoolByteBufAllocator:池化了ByteBuf的实例以提高性能并最大限度地减少内存碎片。实现使用了jemalloc的高效方法来分配内存。
	3. UnpooledByteBufAllocator：每次返回一个新实例。
2. Unpooled缓冲区
	1. 提供了静态的辅助方法来创建未池化的ByteBuf实例。
	2. 非网络组件。
3. ByteBufUtil类
	1. 通用API，静态辅助方法。外部实现。
	2. hexdump()以16进制打印ByteBuf内容
### 5.6 引用计数 ###
1. 引用计数:一种通过在某个对象所持有的资源不再被其他对象引用时释放该对象所持有的资源来优化内存使用和性能的技术。
2. 实现了ReferenceCounted。
3. 背后涉及到跟踪到某个特定对象的活动引用数量。

## 第6章 ChannelHandler和ChannelPipeline ##
### 6.1 ChannelHandler ###
1. Channel生命周期
	1. Channel已经被定义，但还未注册到EventLoop,Channel处于活动状态，可以接收和发送数据，没有连接到远程节点。
	2. 正常生命周期:ChannelRegistered--->ChannelActive--->ChannelInactive--->ChannelUnregistered。
2. ChannelHandler生命周期
	1. 在添加或者从ChannelPipeline移除这些操作的方法中都会接受一个ChannelHandlerContext参数。
	2. 生命周期方法
		1. handlerAdded():当把ChannelHandler添加到ChannelPipeline中时被调用。
		2. handlerRemoved():当从ChannelPipeline中移除ChannelHandler时被调用。
		3. exceptionCaught():当处理过程中在ChannelPipeline中有错误产生时被调用。
	3. ChannelHandler的两个重要子接口
		1. ChannelInboundHandler
		2. CHannelOutboundHandler
3. ChannelInboundHandler接口
	1. channelRegistered():已经注册到EventLoop并且能够处理I/O时被调用。
	2. channelUnregistered():从EventLoop注销并且无法处理任何I/O时被调用。
	3. channelActive():处于活动状态被调用，并且已经连接绑定并且就绪。
	4. channelInactive():离开活动状态并且不再连接它的远程节点时被调用。
	5. channelReadComplete():当上一个读操作完成时被调用。
	6. channelRead():读取数据时被调用。
	7. channelWritablityChanged():写状态发生变化时被调用。
	8. userEventTriggered();
	9. 当某个ChannelInboundHandler的实现重写了channelRead()方法时，此时就会负责显示的释放和池化的ByteBuf实例相关的内存。Netty提供了ReferenceCountUtil.release();
4. ChannelOutboundHandler
	1. 出站操作和数据将由ChannelOutboundHandler处理。它的方法将被Channel,ChannelPipeline以及ChannelHandlerContext调用。
	2. ChannelOutboundHandler的可以按需推迟操作或者事件，从而可以通过一些复杂的方法来处理请求。
	3. bind():当请求将channel绑定到本地地址时被调用。
	4. connect():当请求将Channel连接到远程节点时被调用；
	5. disconnect():当请求将Channel从远程节点断开时被调用；
	6. close():关闭请求；
	7. deregister():当请求将Channel从EventLoop注销时被调用；
	8. read():当请求通过Channel读取更多数据时被调用；
	9. flush():当请求通过Channel将入队数据冲刷到远程节点时被调用；
	10. write():当请求通过Channel将数据写入到远程节点时被调用。
	11. ChannelPromise与ChannelFuture
5. ChannelHandler适配器
	1. 可以使用ChannelInboundHandlerAdapter和ChannelOutboundAdapter作为ChannelHandler起点。这两个适配器分别提供了ChannelInboundHandler和ChannelOutboundHandler的基本实现。通过扩展抽象类ChannelHandlerAdapter。获得共同的超接口ChannelHandler的方法。
	2. 提供的方法体调用了相关联的ChannelHandlerContext,从而将事件转发到ChannelPipeline中的下一个ChannelHandler中。
6. 资源管理
	1. 当调用ChannelInboundHandler.channelRead()或者ChannelOutboundHandler.write()处理数据时，需要保证资源不会泄露。netty中通过引用计数来处理池化的ByteBuf。
	2. Netty提供ResourceLeakDetector来检测内存泄漏。
	3. 内存泄露级别：
		1. disabled 禁用泄露检测
		2. simple 使用1%的默认采样率检测并报告任何发现的泄露，这是默认级别。
		3. advance 使用默认采样率，报告所发现的任何泄露以及对应的消息被访问的位置。
		4. paranoid 每次访问都会堆消息进行采样。
### 6.2 ChannelPipeline接口 ###
1. ChannelPipeline是一个拦截流经Channel的入站和出站事件的ChannelHandler实例链。
2. 每一个新创建的Channel都会被分配到一个新的ChannelPipeline，Channel不能附加到另一个ChannelPipeline，也不能分离其当前的。
3. 根据事件的起源，事件将会被ChannelInboundHandler或者ChannelOutboundHandler处理，随后调用ChannelHandlerContext实现，他将转发给同一个超类型的下一个ChannelHandler。
4. ChannelHandlerContext
	1. ChannelHandlerContext使得ChannelHandler能够和它的ChannelHandler以及其他的ChannelHandler交互。ChannelHandler可以通知其所属的ChannelPipeline中的下一个ChannelHandler,甚至动态修改所属的ChannelPipeline。
	2. ChannelHandlerContext具有丰富的用于处理事件和执行I/O操作的API。
5. 修改ChannelPipeline
	1. ChannelPipeline的API公开了用于调用入站和出站操作的附加方法。用于通知ChannelInboundHandler在ChannelPipeline中所发生的事件。
	2. ChannelPipeline保存了与Channel相关联的ChannelHandler；
	3. ChannelPipeline可以根据需要，通过添加或删除ChannelHandler来动态修改；
	4. ChannelPipeline有丰富的API调用，以响应入站和出出站事件。
### 6.3 ChannelHandlerContext接口 ###
1. 代表了ChannelHandler和ChannelPipeline之间的关联，有ChannelHandler添加到ChannelPipeline中时，都会创建ChannelHandlerContext。
2. 主要功能是管理他所关联的ChannelHandler和在同一个ChannelPipeline中的其他ChannelHandler之间的交互。

## 第7章 EventLoop和线程模型 ##
1. 线程模型
	1. 指定了操作系统，编程语言，框架或应用程序上下文中的线程管理关键方面。何时以及如何创建线程将对应用程序代码执行产生显著的影响。
	2. 基本的线程池化模式：
		1. 从池的空闲线程列表中选择一个Thread,并且指派它去运行一个已提交的任务。
		2. 当任务完成时，该Thread返回给该列表，使其可以被重用。
2. EventLoop接口
	1. 运行任务来处理在连接生命周期内发生的事件是任何网络框架的基本功能。---事件循环。
3. 任务调度
4. 实现细节
	1. Netty性能的卓越性取决于当前执行的Thread的身份确定。确定是否是分配给当前Channel以及它的EventLoop的那个线程。负责处理Channel的整个生命周期内的所有事件。
	2. 如果调用线程正式支撑EventLoop的线程，那么所提交的代码块将会被执行。否则EventLoop将调度该任务以便稍后执行，并将它放入到内部队列中。当EventLoop下次处理它的事件时，它会执行任务队列中那些任务/事件。
	3. EventLoop调度：
		1. 将要在EventLoop执行的任务；
		2. 在把任务传递给execute方法之后，执行检查以确定当前线程是否就是分配给EventLoop的那个线程。
		3. 如果就是相同的线程，则你在EventLoop中，可以直接执行任务。
		4. 如果线程不是EventLoop那个线程，则将任务放入队列以便EventLoop下一次处理它的事件时执行。
5. EventLoop/线程的分配
	1. 服务于Channel的I/O和事件的EventLoop包含在EventLoopGroup中，根据不同的传输实现，EventLoop的创建和分配方式不同。
	2. 异步传输
	3. 阻塞传输
## 第8章 引导 ##
1. 引导一个程序是指：对她进行配置，并使其运行起来的过程。
### 8.1 Bootstrap类 ###
1. 包括一个抽象的父类和两个具体的引导子类。服务器致力于使用一个父Channel来接受来自客户端的连接，并创建子Channel以用于他们之间通信。
2. 客户端将最可能只需要一个单独的，没有父channel的channel来用于所有网络交互。
3. 引导类:Cloneable
	1. 有时候创建需要多个具有类似配置或则完全相同配置的Channel。为了支持这种模式而又不需要为每个Channel都创建并配置一个新的引导类实例，AbstractBootstrap被标记为Cloneable。在一个已经配置完成的引导类实例上调用clone()方法将返回另一个可以立即使用的引导类实例。
	2. 这是浅拷贝。后者将在所有克隆Channel实例之间共享。
4. 引导客户端和无连接协议

### 8.2 引导客户端和无连接协议 ###
1. group():用于处理Channel所有事件得EventLoopGroup;
2. channel():
3. Bootstrap类会在bind()方法被调用后创建一个新的Channel,在这之后调用connect()方法建立连接。

### 8.3 引导服务器 ###
1. ServerBootstrap类
	1. group():设置ServerBootstrap要用得EventLoopGroup,用于ServerChannel和被接受的子Channel的I/O处理。
	2. channel():
	3. channelFactory()
	4. bind():绑定ServerChannel并返回一个ChannelFuture,其将会在绑定操作完成后收到通知。
2. 引导服务器