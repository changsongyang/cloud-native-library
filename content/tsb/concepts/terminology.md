---
title: 术语
description: "用于描述 TSB 及其环境的数据表。"
weight: 9
---

### Tetrate Service Bridge (TSB)

#### Tetrate Service Bridge (TSB)

TSB 是一个服务网格管理平面，旨在提供对基础设施的集中控制。它映射到你的组织结构，使你能够管理资源、访问权限、网络和安全性。

#### 组织

组织代表管理共享基础设施的公司，由多个租户和工作区组成。

#### 租户

租户是组织内共享资源和访问权限的子组，例如团队或部门。

#### 工作空间

工作区是团队管理其命名空间的定义区域。它隔离与跨不同集群的特定团队的命名空间相关的配置。

#### 服务

具有唯一身份和身份验证的网络可访问目的地。

#### 团体

工作区中资源的逻辑分组，分类为网关、流量或安全组。

#### 应用

公开 API 的服务的逻辑分组，在租户的工作区中进行管理。它使开发人员能够配置服务行为和访问条件。

#### 应用程序接口

应用程序公开的一组端点，使开发人员能够使用 OpenAPI 文档配置行为。

#### 用户

与 TSB 相关的实体，包括人类和非人类实体。可以将用户组织成团队并通过 RBAC 分配访问权限。

#### 团队

一组用户使用 RBAC 分配对资源的访问权限。建议团队进行访问分配，增强安全性。

### 架构组件

#### 管理平面

在 TSB 环境中配置网络、安全性和可观察性的主要界面。

#### 全局控制平面/XCP

管理平面的一部分，负责多集群功能和状态管理。

#### 控制平面

每个集群中部署的本地 Istio 服务网格用于管理集群内的网络、流量路由和安全性。

#### 数据平面

由 Envoy 提供支持，使用 sidecar 代理促进服务之间的数据传输。

#### Sidecar 代理

Envoy 实例与应用程序一起部署，处理流量管理、安全性和可观察性。

#### 负载均衡器

根据可用性和容量在服务器之间分配传入请求。

#### 网关

位于网格边缘的负载均衡器，用于传入/传出 HTTP/TCP 连接。

#### 服务注册中心

中心点列出了 TSB 已接入集群中的每项服务。

#### Front Envoy（Envoy 网关）

Envoy 网关接受 TSB 组件的传入流量。

### 其他术语

#### Kubernetes

开源容器编排平台。

#### 集群

一组包含 Kubernetes Pod、VM 或裸机资源的计算节点。

#### 命名空间

Kubernetes 对资源进行分组，逻辑地组织容器、pod 或节点。

#### 故障域

当关键组件出现问题时，环境的一部分容易发生故障。

#### 筒仓

一组故障域形成一个推理和复制单元。

#### 可用区

云提供商区域中用于部署资源的隔离位置，受相关故障模式的影响。

#### 地区

物理位置包括数据中心，包含可用区域。

### 开源软件项目

#### Istio

提供网络自动化和灵活性的开源服务网格。

#### Envoy

适用于现代架构的 L7 代理和通信系统。

#### SkyWalking

提供分布式跟踪和指标聚合的可观察性应用程序平台和 APM 工具。

#### 下一代访问控制 (NGAC)

增强传统 ABAC 的基于图形的访问控制系统。