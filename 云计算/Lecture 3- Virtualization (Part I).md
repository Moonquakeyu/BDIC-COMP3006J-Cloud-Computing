  

## 1. 虚拟化的动机 (Motivation for Virtualization)

### 1.1 实际问题

- 当尝试安装不兼容的应用时遇到的困难
- 反复在系统上安装应用程序的时间成本
- 传统企业IT模式：每次需要新应用程序时购买新服务器，导致资源浪费

### 1.2 系统负载增加时的解决方案

- **Vertical scaling** (scaling up)：通过添加更多CPU、内存等服务器组件来垂直扩展
- **Horizontal scaling** (scaling out)：通过设置额外服务器并运行应用程序的多个副本来水平扩展
- 水平扩展在硬件成本上相对较低，但可能需要对应用程序代码进行重大更改
- 某些应用组件（如关系型数据库）很难进行水平扩展

### 1.3 云环境需求

在云环境中，我们需要一个**managed execution environment**，可以提供：

- 增强的安全性
- 更广泛的功能，如共享(share)、聚合(Aggregation)、模拟(Emulation)和隔离(Isolation)

## 2. 虚拟化简介 (Introduction to Virtualization)

### 2.1 基本概念

- **Virtualization**抽象了**计算机和通信系统的底层物理资源 (abstract the underlying physical resource)**，简化了它们的使用
- 它将用户**相互隔离，支持复制**，从而增加**系统的弹性和可靠性** ( isolate user form one another and supports replication)
- 虚拟化技术不仅为执行应用程序提供虚拟环境，还为存储、内存和网络提供虚拟环境
- 虚拟化通常与**hardware virtualization (硬件虚拟化)**同义，这是高效提供基础设施即服务(IaaS infrastructure-as-a-Service)解决方案的基础

### 2.2 虚拟化的驱动因素

1. 提高性能和计算能力 (Increased performance and computing capacity)
2. 硬件和软件资源未充分利用 (Underutilized hardware and software resources)
3. 空间不足 (Lack of space)
4. 绿色计划 (Greening initiatives)
5. 管理成本上升 (Rise of administrative costs)

## 3. 虚拟化系统和机器参考模型 (Virtualization System and Machine Reference Model)

### 3.1 虚拟化参考模型的三个主要组件

1. **Guest**：与虚拟化层交互而非与主机交互的系统组件
2. **Host**：客户应该被管理的原始环境
3. **Virtualization layer**：负责重建客户将操作的相同或不同环境的层
    
    ![[截屏2025-04-07_12.45.04.png]]
    

### 3.2 受管执行 (Managed Execution) 的特性

虚拟化执行环境提供的功能：

- **Sharing**（共享）：允许在同一主机内创建单独的计算环境，充分利用强大客户的能力
- **Aggregation**（聚合）：将一组单独的主机绑定在一起，向客户展示为单个虚拟主机
- **Emulation**（模拟）：客户程序在虚拟化层控制的环境中执行
- **Isolation**（隔离）：为客户（操作系统、应用程序或其他实体）提供完全独立的执行环境

### 3.3 增强的安全性

- 以完全透明的方式控制客户执行的能力，为提供安全、受控的执行环境开辟了新可能
- 这种间接层允许虚拟机管理器控制和过滤客户活动，从而防止执行某些有害操作
- 主机暴露的资源可以对客户隐藏或受到保护

### 3.4 机器参考模型 (Machine Reference Model)

- 在计算堆栈的不同级别虚拟化执行环境需要一个参考模型，定义抽象级别之间的接口
- 虚拟化技术实际上替换了其中一个层，并拦截指向它的调用
- 层之间的清晰分离简化了它们的实现，只需要模拟接口并与底层层适当交互
    
    ![[截屏2025-04-07_12.56.12.png]]
    

### 3.5 机器参考模型的层次结构

- 最底层：Instruction Set Architecture (ISA)定义处理器、寄存器、内存和中断管理的指令集
- ISA是硬件和软件之间的接口，对操作系统开发者(System ISA)和直接管理底层硬件的应用开发者(User ISA)很重要
- **Application Binary Interface (ABI)** 将操作系统层与应用程序和库分开
- ABI涵盖低级数据类型、对齐和调用约定等细节，定义可执行程序的格式
- 系统调用在此级别定义，这个接口允许应用程序和库在实现相同ABI的操作系统之间移植
- 最高抽象级别由**Application Programming Interface (API)**表示，它将应用程序接口到库和/或底层操作系统
    
    ![[截屏2025-04-07_12.56.50.png]]
    

## 4. 虚拟机类型 (Types of Virtual Machines)

### 4.1 虚拟机的基本定义

- *Virtual machine (VM)**是一个隔离环境，看起来像是一台完整的计算机，但实际上只能访问计算机资源的一部分
    
    ![[截屏2025-04-07_13.00.25.png]]
    

### 4.2 两种主要类型的虚拟机

1. **Process VM**：为单个进程创建的虚拟平台，进程终止后销毁
2. **System VM**：支持一个操作系统以及多个用户进程

### 4.3 系统级虚拟化

- **OS-level virtualization**：使物理服务器能够运行多个隔离的OS实例，这些实例被称为**containers**或**Virtual Private Servers**
- **Virtual Machine Monitor (VMM/hypervisor)**：使多个VM能够共享一个物理系统，将计算机系统的资源分区为一个或多个虚拟机

### 4.4 Hypervisor在VM架构中的角色

三种主要的hypervisor类型：

![[截屏2025-04-07_13.33.30.png]]

1. **Traditional VM/"bare metal" hypervisor**：直接在主机硬件上运行的薄软件层，主要优势是性能（例如：VMware ESX, Xen）
2. **Hybrid**：hypervisor与现有OS共享硬件
3. **Hosted**：VM在现有OS之上运行，主要优势是VM更容易构建和安装

## 5. 虚拟化与云计算 (Virtualization and Cloud Computing)

### 5.1 虚拟机迁移

![[截屏2025-04-07_13.36.53.png]]

- **Virtual machine migration**：虚拟机实例的移动
- 可以通过两种主要方式进行：
    1. 暂时停止其执行并将其数据移至新资源
    2. **Live migration**：在运行时移动实例，实现更高效但实现更复杂的迁移

## 6. 云虚拟化的优缺点 (Pros and Cons of Cloud Virtualization)

### 6.1 虚拟化的优势

- **Managed execution**和**isolation**是虚拟化最重要的优势，允许构建安全和可控的计算环境
- **Portability**（可移植性）是虚拟化的另一个优势，特别是对于执行虚拟化技术而言
- **Portability**和**self-containment**（自包含性）有助于降低维护成本
- 使用虚拟化可以实现更高效的资源使用，多个系统可以安全地共存并共享底层主机的资源

### 6.2 虚拟化的缺点

- **Performance Degradation**（性能下降）：由于虚拟化在客户和主机之间插入了一个抽象层，客户可能会遇到增加的延迟和延迟
- **Inefficiency and Degraded User Experience**（低效率和用户体验下降）：虚拟化有时会导致主机的低效使用，特别是主机的一些特定功能可能无法通过抽象层暴露
- **Security Holes and New Threats**（安全漏洞和新威胁）：虚拟化为一种新的意外形式的网络钓鱼打开了大门

## 7. 容器 (Containers)

### 7.1 容器的基本概念

- **Container**大致类似于VM，主要区别在于每个容器不需要自己的完整OS
- 单个主机上的所有容器共享一个操作系统，释放大量系统资源（CPU、RAM、存储）
- 容器启动快速且超便携，将容器工作负载从笔记本电脑移至云端再到数据中心的VM或裸机轻而易举

### 7.2 容器与VM的比较

- 所有容器在主机OS中运行的完全相同的内核上执行系统调用
- 这个单一内核是唯一在主机CPU上执行x86指令的内核
- CPU不需要进行任何类型的虚拟化，就像使用VM那样
    
    ![[截屏2025-04-07_13.46.32.png]]
    

### 7.3 容器隔离机制

两种机制使容器隔离成为可能：

1. **Linux Namespaces**：确保每个进程看到系统的个人视图（文件、进程、网络接口、主机名等）
2. **Linux Control Groups (cgroups)**：限制进程可以消耗的资源量（CPU、内存、网络带宽等）

### 7.3.1 使用Linux命名空间隔离进程

- 默认情况下，每个Linux系统最初有一个单一的命名空间
- 所有系统资源（如文件系统、进程ID、用户ID、网络接口等）属于这个单一命名空间
- 我们可以创建额外的命名空间并在它们之间组织资源
- 运行进程时，在其中一个命名空间中运行它，该进程将只看到同一命名空间中的资源

### 7.3.2 Linux命名空间的类型

![[截屏2025-04-07_13.50.38.png]]

存在多种命名空间：

- **Mount (mnt)**
- **Process ID (pid)**
- **Network (net)**
- **Inter-process communication (ipc)**
- **UTS**
- **User ID (user)**

例如：UTS命名空间决定了在该命名空间内运行的进程看到的主机名和域名

### 7.3.3 通过限制系统资源进行容器隔离

- 容器隔离的另一半是限制容器可以消耗的系统资源量（**Linux Control Groups**）
- 这是通过**cgroups**实现的，它限制进程（或进程组）的资源使用
- 进程不能使用超过配置的CPU、内存、网络带宽等量

## 8. Docker

### 8.1 Docker的定义

- **Docker**是一个用于打包、分发和运行应用程序的平台
- 它允许你将应用程序与其整个环境一起打包
- 这可以是应用程序所需的一些库，或者通常在已安装操作系统的文件系统上可用的所有文件
- Docker使将此包传输到中央存储库成为可能，从中可以传输到任何运行Docker的计算机并在那里执行

### 8.2 Docker的三个主要概念

1. **Images**：基于Docker的容器镜像，打包应用程序及其环境，包含将可用于应用程序的文件系统和其他元数据
2. **Registries**：Docker Registry是一个存储Docker镜像的存储库，便于在不同人和计算机之间轻松共享这些镜像
3. **Containers**：基于Docker的容器是从基于Docker的容器镜像创建的常规Linux容器

### 8.3 Docker镜像的构建、分发和运行流程

Docker的工作流程包括构建镜像、将镜像推送到注册表，以及在不同机器上拉取和运行

![[截屏2025-04-07_13.55.50.png]]

### 8.4 Docker容器与虚拟机的比较

- VM方式：在三个VM上运行六个应用
- Docker方式：直接在Docker容器中运行六个应用这种比较突显了Docker容器的资源效率

### 8.5 理解镜像层

![[截屏2025-04-07_13.56.30.png]]

- Docker镜像由层组成
- 不同镜像可以包含完全相同的层，因为每个Docker镜像都是建立在另一个镜像之上的
- 这加速了镜像在网络上的分发
- 层不仅使分发更高效，还有助于减少镜像的存储占用
- 容器镜像层是只读的；运行容器时，在镜像层之上创建一个新的可写层

### 8.6 容器镜像的可移植性限制

- 理论上，Docker容器镜像可以在任何运行Docker的Linux机器上运行，但存在一个小问题
- 所有在主机上运行的容器都使用主机的Linux内核
- 如果容器化应用程序需要特定的内核版本，它可能无法在每台机器上工作
- 如果机器运行不同版本的Linux内核或没有相同的内核模块可用，应用程序就无法在其上运行
- 虽然容器比VM轻量得多，但它们对其中运行的应用程序施加了某些限制

## 9. 总结

虚拟化技术是云计算的基础，通过抽象物理资源、提供隔离环境、支持资源共享和聚合，使得云服务能够提供灵活、可扩展的计算能力。虚拟机和容器是两种主要的虚拟化技术，各有优缺点：虚拟机提供更完整的隔离但资源开销较大，容器则轻量快速但共享主机内核。Docker作为容器技术的代表，通过镜像、注册表和容器三个核心概念，简化了应用程序的打包、分发和运行流程。