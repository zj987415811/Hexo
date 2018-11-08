---
title: k8s系列之k8s核心原理
date: 2018-09-29 14:22:49
tags: k8s
---
k8s核心原理介绍
<!-- more -->

## 第2章 Kubernetes核心原理 ##
### 1.Kubernetes API Server分析 ###
1. 功能和地位	
- 提供了集群管理的API接口；
- 成为集群内各个功能模块之间数据交互和通信的中心枢纽；
- 拥有完备的集群安全机制。
2. 如何访问K8S API
	1. K8S API通过K8S apiserver进程提供服务，运行于master节点。该进程有2个端口：
		1. 本地端口
			1. 该端口用于接收Http请求；
			2. 默认值为8080，可以通过修改--insecure-port值来修改默认值。
			3. 默认ip地址是localhost,通过修改--insecure-bing-address的值来修改该IP地址；
			4. 非认证或授权的Http请求通过该端口访问API Server。
		2. 安全端口
			