# Google Borg集群管理系统详细笔记

## 1. Borg系统概述 (System Overview)

Borg是Google开发的大规模集群管理系统，负责运行数十万个任务，来自数千个不同应用程序，跨越多个集群，每个集群包含多达数万台机器。Borg系统是Google内部最核心的基础设施之一，几乎所有Google的应用和服务都运行在Borg上。

### 1.1 主要优势 (Main Benefits)

- **隐藏资源管理和故障处理细节**：使用户可以专注于应用程序开发
- **提供高可靠性和高可用性**：支持应用程序实现相同的可靠性目标
- **高效管理资源**：能够有效地在成千上万台机器上运行工作负载

## 2. 架构组件 (Architecture Components)

### 2.1 整体架构 (Overall Architecture)

Borg系统的主要组件包括：

- **Borgmaster**：集群的中央控制器
- **Borglet**：每台机器上运行的agent
- **Cell**：由Borgmaster管理的一组机器

### 2.2 Borgmaster

**Borgmaster**是Borg系统的核心控制器，由两个主要进程组成：

- **主Borgmaster进程**：
    
    - 处理客户端RPC请求（创建作业、查找作业等）
    - 管理系统中所有对象（机器、任务、allocs等）的状态机
    - 与Borglet通信
    - 提供Web UI（作为Sigma的备份）
- **Scheduler**：负责为任务分配机器
    

Borgmaster采用了**高可用性设计**：

- 采用**primary-backup approach with Paxos-based leader election**（基于Paxos的主备模式和领导者选举）
- 逻辑上是单一进程，实际上有5个副本
- 每个副本在内存中维护集群状态的副本
- 状态也记录在基于Paxos的分布式存储中
- 一个选举出的master作为Paxos的leader和状态变更者
- 选举新master通常需要10秒左右，但大型cell可能需要一分钟
- 使用checkpoint机制记录状态和变更

### 2.3 **Scheduler** (调度器)

**Scheduler**是**负责做出调度决策的关键组件**，它负责将任务分配到满足其资源需求和约束条件的机器上。

调度算法包括两部分：

1. **可行性检查 (Feasibility Checking)**：找到任务可以运行的机器集合
2. **评分 (Scoring)**：从可行的机器中选择一个最佳的

Borg使用**two-phase scheduling mechanism**（两阶段调度机制）：

- 首先检查机器是否满足任务的约束条件和资源需求
- 然后对可行的机器进行评分和排序
- 选择得分最高的机器

为了提高可扩展性，调度器使用了几种优化技术：

- **Score caching**：缓存评分结果，直到机器或任务的属性发生变化
- **Equivalence classes**：对具有相同需求的任务分组，只为每个等价类评估一次
- **Relaxed randomization**：随机检查一部分机器而非全部，找到足够多的可行机器后停止

### 2.4 Borglet

**Borglet**是运行在每台机器上的Borg代理程序，负责：

- 启动和停止任务
- 重启失败的任务
- 通过操作内核设置管理本地资源
- 滚动调试日志
- 向Borgmaster和其他监控系统报告机器状态

Borgmaster定期轮询每个Borglet以获取机器当前状态并发送待处理的请求。如果Borglet不响应多个轮询消息，其机器将被标记为停机，原有任务会被重新调度到其他机器上。

### 2.5 Cell (单元)

**Cell**是由Borgmaster管理的一组机器，通常有数千至上万台机器。一个Cell内的机器属于同一个集群，由高性能数据中心级网络连接。

Cell的特点：

- 中等规模的Cell约有10,000台机器
- Cell中的机器在多个维度上是异构的（CPU、RAM、磁盘、网络、处理器类型等）
- 一个集群通常有一个大型Cell和几个小型测试或特殊用途的Cell
- Cell设计避免单点故障

## 3. 工作负载和任务管理 (Workload & Task Management)

### 3.1 工作负载类型 (Workload Types)

Borg管理两种主要的工作负载：

1. **长期运行的服务 (Long-running services)**：
    
    - 面向终端用户的产品（Gmail、Google Docs、网络搜索）
    - 内部基础设施服务（BigTable等）
    - 具有延迟敏感性，需要"永不"宕机
    - 被分类为"production"（prod）工作负载
2. **批处理作业 (Batch jobs)**：
    
    - 完成时间从几秒到几天不等
    - 对短期性能波动不敏感
    - 被分类为"non-production"（non-prod）工作负载

### 3.2 Jobs和Tasks

在Borg中，工作组织为：

- **Job**：一组运行相同程序的tasks，具有名称、所有者和任务数量等属性
- **Task**：Job中的单个实例，映射到机器上容器中运行的一组Linux进程
- **Alloc**：机器上预留的资源集，一个或多个任务可以在其中运行
- **Alloc set**：类似于Job，是跨多台机器的alloc组

任务可以指定各种属性：

- 资源需求（CPU、内存、磁盘空间等）
- 约束条件（硬约束或软约束）
- 任务在Job中的索引
- 命令行参数等

任务生命周期状态转换： ![任务状态转换图](https://i.imgur.com/example2.png)

## 4. 资源管理 (Resource Management)

### 4.1 Priority (优先级)和Quota (配额)

Borg使用**优先级和配额**来管理资源分配和准入控制：

- **Priority**（优先级）：
    
    - 每个Job都有一个优先级（小的正整数）
    - <font color="#ff0000">高优先级任务可以抢占低优先级任务的资源</font>
    - 优先级带（按降序）：monitoring、production、batch、best effort
    - Production优先级带内的任务不能互相抢占，以避免抢占级联
- **Quota**（配额）：
    
    - 决定哪些Job可以被接纳进行调度
    - 表示为特定优先级下的资源数量向量（CPU、RAM、磁盘等）
    - 更高优先级的配额成本更高
    - 配额检查是准入控制的一部分，而非调度

### 4.2 细粒度资源请求 (Fine-grained Resource Requests)

Borg允许用户以**细粒度**方式请求资源：

- CPU以milli-cores（千分之一核）为单位
- 内存和磁盘空间以字节为单位
- 不强制使用固定大小的容器或虚拟机
- 每个资源维度（CPU核心、RAM、磁盘空间、磁盘访问率、TCP端口等）可以独立指定

### 4.3 资源回收 (Resource Reclamation)

Borg使用**资源回收**机制提高集群利用率：

- 用户通常请求比实际使用更多的资源（为了应对峰值）
- Borg会估计任务实际将使用的资源量（称为**reservation**预留）
- 回收未使用的资源给可以容忍低质量资源的工作（如批处理作业）
- Borgmaster每几秒钟根据Borglet捕获的细粒度使用信息计算预留量
- 调度器为prod任务使用limits（上限），为non-prod任务使用reservations（预留）

资源回收的效果：

- 大约20%的工作负载运行在回收的资源中
- 显著提高了集群利用率
- 如果预测错误，会优先杀死non-prod任务而非prod任务

## 5. 隔离机制 (Isolation)

### 5.1 安全隔离 (Security Isolation)

Borg使用多种安全隔离机制：

- **Linux chroot jail**：作为主要安全隔离机制
- **borgssh命令**：提供受限SSH访问
- 对外部软件，使用**VM和安全沙箱**技术

### 5.2 性能隔离 (Performance Isolation)

Borg使用多种技术实现性能隔离：

- **Linux cgroup-based resource container**：所有Borg任务都在基于cgroup的资源容器中运行
- **Application classes** (应用类别)：
    - **Latency-sensitive (LS)** 延迟敏感型：面向用户的应用和共享基础设施服务
    - **Batch** 批处理型：其他所有任务
- **资源类别**：
    - **可压缩资源**（CPU周期、磁盘I/O带宽）：可以降低服务质量回收
    - **不可压缩资源**（内存、磁盘空间）：通常只能通过终止任务回收
- **CPU调度优化**：
    - 延迟敏感任务可以预留整个物理CPU核心
    - 使用定制的Linux CPU调度器（CFS）支持低延迟和高利用率
    - 动态调整贪婪LS任务的资源上限

## 6. 可用性设计 (Availability Design)

### 6.1 高可用性策略

Borg采用多种技术提高可用性：

- 自动重新调度被驱逐的任务
- 将Job的任务分散在不同故障域（机器、机架、电源域）
- 限制维护活动中的任务中断率
- 使用声明式期望状态和幂等变更操作
- 限制寻找不可达机器上任务的新位置的速率
- 避免重复导致任务或机器崩溃的任务-机器配对
- 通过反复重新运行logsaver任务恢复写入本地磁盘的关键中间数据

可用性的核心设计：**已运行的任务在Borgmaster或Borglet宕机时继续运行**

## 7. 集群效率 (Cluster Efficiency)

### 7.1 共享策略 (Sharing Policies)

Borg的共享设计提高了资源利用率：

- **混合工作负载**：几乎所有机器同时运行prod和non-prod任务
- **资源共享**：通过资源回收利用预留但未使用的资源
- **大型Cell**：减少资源碎片化，允许运行大型计算
- **用户共享**：不同用户的任务在同一机器上运行

### 7.2 提高效率的关键技术

- **资源回收**：回收未使用的已分配资源
- **细粒度资源请求**：避免资源浪费
- **混合调度策略**：减少资源碎片
- **缓存和等价类**：提高调度器性能
- **大型共享集群**：减少整体资源需求

## 8. 经验教训 (Lessons Learned)

### 8.1 需要避免的设计 (Design to Avoid)

- **Jobs作为唯一的任务分组机制过于限制**：Kubernetes使用labels替代
- **每台机器一个IP地址带来复杂性**：Kubernetes为每个pod和服务提供自己的IP地址
- **为高级用户优化，牺牲普通用户体验**：需要更好的自动化和用户体验

### 8.2 良好的设计经验 (Good Design Experiences)

- **Allocs抽象非常有用**：Kubernetes中的pod是等效概念
- **集群管理不仅仅是任务管理**：需要命名、负载均衡等服务
- **内省能力至关重要**：用户需要访问调试信息
- **Master是分布式系统的核心**：微服务架构更灵活

## 9. 关键术语表 (Key Terminology)

- **Borg**: Google的大规模集群管理系统
- **Borgmaster**: 集群的中央控制器
- **Borglet**: 每台机器上运行的Borg代理程序
- **Cell**: 由Borgmaster管理的一组机器
- **Job**: 运行相同程序的任务集合
- **Task**: Job的单个实例
- **Alloc**: 机器上预留的资源集
- **Priority**: 任务的重要性级别
- **Quota**: 用户可以请求的资源限制
- **Resource Reclamation**: 回收分配但未使用的资源
- **Application Class**: 任务类别（延迟敏感型vs批处理型）