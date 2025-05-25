## 1. Docker 容器回顾

### 1.1 Docker 容器概述

- Docker 容器允许在同一主机上运行多个隔离的应用程序
- 每个容器包含自己的应用程序、**binaries**和**libraries**
- 所有容器共享同一个**host OS**，使得资源利用更加高效

### 1.2 Docker 镜像的生命周期

Docker 镜像的构建、分发和运行过程包括以下步骤：

1. 开发者指示 Docker **build**和**push image**
2. Docker 构建镜像
3. Docker 将镜像推送到**registry**
4. 开发者告诉生产环境中的 Docker 运行镜像
5. Docker 从**registry**拉取镜像
6. Docker 从镜像运行容器

## 2. 运行容器 (Running Containers)

### 2.1 基本命令

运行容器的基本命令是：`docker run <image>`

这个应用程序是一个单一的可执行文件（busybox），但它也可能是一个非常复杂的应用程序，有大量的依赖关系。

示例：运行 busybox 镜像

```Shell
$ docker run busybox echo "Hello world"
```

当运行此命令时，Docker 会：

1. 检查 busybox 镜像是否已在本地存储
2. 如果不可用，从**Docker Hub**下载镜像
3. 在隔离的容器中运行指定命令
    
    ![[截屏2025-04-07_14.07.43.png]]
    

### 2.2 容器运行的优势

- 无需安装应用程序或其依赖项，整个应用都被下载并一键执行
- 应用程序在完全隔离的容器中执行，与机器上的其他进程完全分离
- 即使是复杂的应用程序也可以轻松部署和运行

### 2.3 镜像版本控制

- Docker 支持使用**tags**来管理同一镜像的多个版本或变体
- 每个变体必须有唯一的**tag**
- 不明确指定 tag 时，Docker 会假设使用**latest** tag
- 引用镜像的格式：`<image>:<tag>`

## 3. 创建容器镜像 (Creating Container Images)

### 3.1 创建一个简单的 Node.js 应用

- 示例：创建一个 Web 应用，接受 HTTP 请求并响应机器的主机名
- 应用启动在端口 8080 上的 HTTP 服务器
- 响应包含状态码 200 OK 和文本 "You've hit "

代码示例 (app.js)：

```JavaScript
const http = require('http');
const os = require('os');
console.log("Kubia server starting...");
var handler = function(request, response) {
    console.log("Received request from " + request.connection.remoteAddress);
    response.writeHead(200);
    response.end("You've hit " + os.hostname() + "\n");
};
var www = http.createServer(handler);
www.listen(8080);
```

### 3.2 创建 Dockerfile

Dockerfile 是一个包含 Docker 构建镜像时执行的指令列表的文件：

```Plain
FROM node:7
ADD app.js /app.js
ENTRYPOINT ["node", "app.js"]
```

Dockerfile 的主要组件：

- **FROM** 行定义基础镜像（在这个案例中使用 node:7）
- **ADD** 行将本地 app.js 文件添加到镜像的根目录
- **ENTRYPOINT** 行定义运行镜像时应执行的命令

### 3.3 构建容器镜像

构建镜像的命令：

```Plain
$ docker build -t kubia .
```

构建过程：

1. Docker 客户端将目录内容上传到 Docker daemon
2. Docker daemon 如果本地没有 node:7 镜像，会从 Docker Hub 拉取
3. 根据 Dockerfile 中的指令构建新镜像
4. 构建完成后，镜像存储在本地，标记为 kubia:latest
    
    ![[截屏2025-04-07_14.29.21.png]]
    

### 3.4 了解镜像层 (Image Layers)

- 镜像不是单一的二进制文件，而是由多个**layers**组成
- 不同镜像可能共享相同的层，使存储和传输镜像更高效
- 构建镜像时，Dockerfile 中的每个命令都会创建一个新层
- 例如，在构建过程中，Docker 会：
    1. 拉取所有基础镜像层
    2. 创建一个新层添加 app.js 文件
    3. 创建另一个层指定运行镜像时应执行的命令
    4. 将最后一层标记为 kubia:latest
        
        ![[截屏2025-04-07_14.38.10.png]]
        

### 3.5 运行容器镜像

运行镜像的命令：

```Plain
$ docker run --name kubia-container -p 8080:8080 -d kubia
```

主要参数：

- **-name**：指定容器名称
- **p 8080:8080**：将本地机器的端口 8080 映射到容器内的端口 8080
- **d**：以分离模式（后台）运行容器

### 3.6 停止和删除容器

```Shell
$ docker stop kubia-container   # 停止容器
$ docker rm kubia-container     # 删除容器
```

停止容器只会停止主进程，容器本身仍然存在，可以通过 `docker ps -a` 查看

### 3.7 有用的 Docker 命令

- `docker version`：查看安装的 Docker 版本
- `docker ps`：列出正在运行的容器
- `docker image ls`：列出所有拉取的镜像
- `docker stop`：通过容器名称或 ID 停止容器
- `docker kill`：通过杀死执行立即停止容器

## 4. Docker 容器与 DevOps

### 4.1 应用程序的一致环境

- 开发环境和生产环境之间的差异是开发人员和运维团队面临的主要问题
- 生产环境由运维团队管理，而开发人员通常自己维护开发环境
- 理想情况下，应用程序应该在开发和生产中运行在完全相同的环境中

### 4.2 DevOps 概念

- **DevOps**是一种实践，让同一个团队既开发应用程序又参与部署和维护
- 这使开发人员更好地理解用户需求和运维团队面临的问题
- 运维团队负责生产部署和硬件基础设施

### 4.3 NoOps

- **NoOps**是一种理想状态，开发人员可以自行部署应用程序，而无需了解硬件基础设施或与运维团队交互
- 仍然需要有人维护硬件基础设施，但理想情况下不需要处理每个应用程序的特殊性
- Kubernetes 使这一切成为可能

## 5. Kubernetes 介绍

### 5.1 Kubernetes 的定义

- **Kubernetes**是一个软件系统，允许在其上轻松部署和管理容器化应用程序
- 它依赖于 Linux 容器的特性运行异构应用程序，而无需了解这些应用程序的内部细节
- Kubernetes 使您能够在成千上万的计算机节点上运行软件应用程序，就好像所有这些节点都是一台巨大的计算机
- 它抽象化了底层基础设施，并通过这样做，简化了开发和操作团队的开发、部署和管理。

### 5.2 Kubernetes 的核心功能

- Kubernetes 由**master node**和任意数量的**worker nodes**组成
- 开发人员向**master**提交应用程序列表，Kubernetes 将它们部署到**worker nodes**集群
    
    ![[截屏2025-04-07_14.51.50.png]]
    
- 应用程序部署到哪个节点对开发人员或系统管理员来说并不重要
- Kubernetes 可以指定某些应用必须一起运行（在同一个**worker node**上），其他应用则可以分散在集群中

### 5.3 Kubernetes 架构

Kubernetes 集群的架构包括：

- **Control Plane (master)**：控制和管理整个 Kubernetes 系统
    - **API Server**：与其他控制平面组件通信的接口
    - **Scheduler**：为应用程序的每个可部署组件分配**worker node**
    - **Controller Manager**：执行集群级功能，如复制组件，跟踪**worker nodes**等
    - **etcd**：可靠的分布式数据存储，持久存储集群配置
        
        ![[截屏2025-04-07_14.54.07.png]]
        
- **Worker Nodes**：运行容器化应用程序的机器
    - **Container Runtime**（Docker 或其他）：运行容器
    - **Kubelet**：与 API 服务器通信并管理节点上的容器
    - **Kubernetes Service Proxy (Kube-proxy)**：在应用程序组件之间负载均衡网络流量

### 5.4 在 Kubernetes 中运行应用程序

![[截屏2025-04-08_09.32.54.png]]

在 Kubernetes 中运行应用程序的步骤：

1. 将应用程序打包成一个或多个容器镜像
2. 将这些镜像推送到**image registry**
3. 向 Kubernetes API 服务器提交应用程序描述
    - 描述包含容器镜像信息
    - 组件之间的关系
    - 哪些组件需要一起运行
    - 每个组件要运行多少副本
    - 哪些组件为内部或外部客户端提供服务

### 5.5 Kubernetes 的优势

- **简化应用部署**
- **更好地利用硬件**
- **健康自检和自我修复**：如果一个实例停止工作，Kubernetes 会自动重启它
- **自动扩展**：可以根据实时指标（如 CPU 负载、内存消耗等）自动调整副本数量
- **保持应用程序运行**：Kubernetes 确保应用程序的部署状态始终与您提供的描述匹配

## 6. 设置 Kubernetes

### 6.1 Kubernetes 集群设置选项

Kubernetes 可以在以下环境中运行：

- 本地开发机器
- 组织自己的机器集群
- 提供虚拟机的云提供商（Google Compute Engine, Amazon EC2, Microsoft Azure 等）
- 管理的 Kubernetes 集群，如 Google Kubernetes Engine

### 6.2 使用 Minikube 运行本地单节点 Kubernetes 集群

- **Minikube**是设置单节点集群的工具，非常适合本地测试 Kubernetes 和开发应用程序

## 7. Pods 概念

### 7.1 Pods 基本概念

- **Pod**是 Kubernetes 中最重要的核心概念
- Pod 是**co-located**的容器组，代表 Kubernetes 中的基本构建块
- Pod 可以只包含单个容器，或多个容器
- Pod 的关键特点是：当 Pod 包含多个容器时，所有容器始终在单个**worker node**上运行 - 永远不会跨多个**worker nodes**
    
    ![[截屏2025-04-08_09.38.27.png]]
    

### 7.2 为什么需要 Pods

- 在一个容器中运行多个进程会导致问题：难以管理日志、重启单个进程等
- 容器设计为每个容器只运行一个进程
- Pod 允许将密切相关的进程绑定在一起，并将它们作为一个单元管理
- Pod 提供了几乎与单容器相同的环境，同时保持了一定程度的隔离

### 7.3 Pod 内容器如何共享 IP 和端口空间

- Pod 中的容器运行在相同的**Network namespace**中
- 它们共享相同的 IP 地址和端口空间
- 同一 Pod 中的进程需要确保不绑定到相同的端口号，否则会发生端口冲突
- Pod 中的所有容器也具有相同的**loopback network interface**
- 一个容器可以通过 localhost 与同一 Pod 中的其他容器通信

### 7.4 Pods 之间的网络

- Kubernetes 集群中的所有 Pod 都位于单一的、共享的网络地址空间中
- 每个 Pod 都可以通过其他 Pod 的 IP 地址访问任何其他 Pod
- Pod 之间不存在 NAT（网络地址转换）网关
- 当两个 Pod 相互发送网络数据包时，它们会在数据包中看到对方的实际 IP 地址

## 8. 总结

Docker 和 Kubernetes 是云计算中虚拟化和容器编排的关键技术。Docker 提供了打包、分发和运行应用程序的平台，将应用程序与其环境一起打包，使其可以在任何运行 Docker 的计算机上执行。Kubernetes 则提供了在大规模集群上部署、管理和扩展容器化应用程序的能力，抽象底层基础设施，简化了开发和运维工作。Pod 作为 Kubernetes 的核心概念，提供了将相关容器组合在一起并作为一个单元管理的机制，确保应用程序组件能够协同工作，同时保持适当的隔离和资源共享。