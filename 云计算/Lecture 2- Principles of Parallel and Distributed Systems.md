## 1. Parallel Processing 和 Distributed Computing 概述

### 1.1 基础概念

- **Parallel processing** 允许我们通过将大问题分解为较小问题并并行解决来处理复杂任务
- **Cloud computing** 与并行和分布式处理密切相关
- 大规模并行处理需要多方面的进步，包括**algorithms**, **programming languages**, **environments**, **computer architecture**等

### 1.2 并行与分布式计算的区别

- **Distributed computing** 是指在多个系统上的并发执行，通常位于不同地点
- **Parallel processing** 是指在具有大量处理器的单一系统上并发执行
- 分布式计算只有在**coarse-grained parallel applications**（并发活动很少相互通信）时才有效

## 2. 细粒度与粗粒度并行性 (Fine-grained vs. Coarse-grained Parallelism)

### 2.1 两种模式的定义

- **Coarse-grained parallelism**：在并发线程通信之前执行大块代码
    - **简单定义**：任务被分成大块，每块可以相对独立完成，很少需要与其他部分通信
    - **通信频率**：处理单元之间只在完成较大工作块后才需要交换信息
    - **工作方式**：像是将一本书的不同章节分配给不同编辑，他们各自独立工作，只在完成各自章节后才需要协调
    - **现实生活例子**：不同部门独立处理一个大项目的不同模块，只在主要节点才需要会面
- **Fine-grained parallelism**：短暂的计算交替出现，线程等待其他线程的消息
    - **简单定义**：任务被分成很小的部分，这些小部分经常需要相互通信或同步
    - **通信频率**：处理单元之间需要频繁交换信息
    - **工作方式**：像是一群工人在做一项复杂的工作，每个人都负责很小的任务，但他们需要经常停下来互相交流以确保工作协调一致
    - **现实生活例子**：一个厨房里多个厨师一起准备一道复杂菜肴，每个人负责不同步骤，但需要频繁协调

### 2.2 应用案例

- **Fine-grained** 示例：涉及线性代数运算的数值计算
- **Coarse-grained** 示例：Web搜索引擎，其中服务器独立处理web索引和搜索查询的不同部分
- 其他适合**Fine-grained**的案例：图像和信号处理（像素或小块级别的并行）
- 其他适合**Coarse-grained**的案例：自动驾驶车辆的不同系统组件独立处理

## 3. 并行任务的通信方式

### 3.1 主要通信机制

- 并发进程或线程使用**message-passing**或**shared memory**进行通信
- **Shared memory** 是分布式共享内存多处理器系统的定义属性
- **Multicore processors**中，每个核心有私有L1和L2缓存，所有核心共享L3缓存

### 3.2 通信机制的可扩展性

- **Shared memory** 不具备可扩展性，在超级计算机和大型集群中很少使用
- **Message passing** 在大规模分布式系统中独占使用
- 调试基于**message-passing**的应用程序比调试**shared-memory**应用容易得多

## 4. Data Parallelism 与 Task Parallelism

### 4.1 数据并行性

- **Data parallelism**：每个核心使用相同数据的不同子集执行相同的任务集
- 也称为**SIMD** (Same Program Multiple Data)
- 示例：将大量图像从一种格式转换为另一种格式 – 103个处理器可以并行处理109张图像
    
    ![[截屏2025-04-07_10.08.24.png]]
    

### 4.2 任务并行性

- **Task parallelism**：将较大任务分解为多个独立子任务，分配给不同核心
- 示例：各种程序可以并行处理传感器数据，提供图像、声音和数据，每个程序负责识别特定异常

## 5. 加速比与 Amdahl's Law

### 5.1 加速比

- **Speed-up (S)** 衡量并行化的有效性：S(N) = T(1) / T(N)
    - T(1) = 顺序计算执行时间
    - T(N) = 执行N个并行计算时的执行时间

### 5.2 Amdahl's Law 与 Gustafson's Law

- **Amdahl's Law**：如果α是顺序程序在不可并行化计算段上花费的运行时间比例，则 S = 1/α
- **Gustafson's Law**：N个并行进程的缩放加速比 S(N) = N - α(N-1)

## 6. Multicore Processor 加速

### 6.1 多核处理器设计

- 多核处理器可以有不同设计：
    - 核心可以相同或不同
    - 可以是少量强大核心或更多数量的较弱核心
- 更多核心会导致高度并行应用的高加速比，而强大的核心有利于高度顺序化的应用

### 6.2 成本效益考虑

- 多核处理器的成本取决于核心数量以及单个核心的复杂性和功率
- 成本效益设计是实现的加速比超过成本

## 7. 通往云计算的路径：超级计算机

### 7.1 现代超级计算机特点

- 现代超级计算机的计算能力来自**architecture**和**parallelism**，而非更快的时钟频率
- 现代超级计算机由大量处理器和核心组成，通过非常快速且昂贵的定制互连通信

### 7.2 超级计算机案例

- **Lawrence Livermore National Laboratory**系统：1,572,864个核心，16.32 PFlops，功耗7.89 MW
- **Sunwai TaihuLight**：10,649,600个核心，125.436 PFlops，功耗15.371 MW
- **Fugaku**：7,299,072个核心，415.530 TFlop/sec，功耗28.3345 MW
- **Oak Ridge National Laboratory**系统：2,414,592个核心，148.600 TFlops/sec，功耗10 MW

## 8. 分布式系统 (Distributed Systems)

### 8.1 定义与特点

- **Distributed system**是由通过网络连接的计算机集合和一种称为**middleware**的分发软件组成，使计算机能够协调活动并共享系统资源
- 用户将系统视为单一的、集成的计算设施

### 8.2 分布式系统的理想属性

- **Access transparency**：使用相同操作访问本地和远程信息对象
- **Location transparency**：访问信息对象而不知道其位置
- **Concurrency transparency**：多个进程可无干扰地并发运行
- **Replication transparency**：信息对象的多个实例提高可靠性
- **Failure transparency**：隐藏故障
- **Migration transparency**：信息对象可移动而不影响其操作
- **Performance transparency**：可根据负载和服务质量要求重新配置系统
- **Scaling transparency**：系统可扩展而不改变结构

### 8.3 分布式系统的主要特征

- 用户将系统视为单一、集成的计算设施
- 组件是自主的
- 调度和其他资源管理由每个系统实施
- 存在多个控制点和多个故障点
- 资源可能不总是可访问的
- 可通过添加资源进行扩展
- 即使在低硬件/软件/网络可靠性水平下也能保持可用性

## 9. 模块化 (Modularity)

### 9.1 基本概念

- **Modularity**是人造系统设计的基本概念；系统由具有明确定义功能的组件或模块构成
- 模块化支持**关注点分离、鼓励专业化、提高可维护性、降低成本并缩短系统开发时间**
- **Modularity**, **layering**和**hierarchy**是应对分布式应用软件复杂性的手段

### 9.2 软模块化 (Soft Modularity)

- **Soft modularity**将程序分为相互调用的模块，使用共享内存或遵循过程调用约定通信
- 隐藏模块实现细节
- 一旦定义了模块接口，模块可以独立开发
- 模块可以用不同编程语言编写并独立测试
- 挑战：增加调试难度、可能有命名冲突和错误的上下文规范、调用者和被调用者在同一地址空间

### 9.3 强制模块化 (Enforced Modularity)

- 客户端-服务器范式（**client-server paradigm**）
- 模块仅通过发送和接收消息进行交互
- 更健壮的设计：客户端和服务器是独立模块，可能分别失败
- 不允许错误传播
- 服务器是无状态的，不必维护状态信息
- 强制模块化降低攻击可能性
- 通常基于**RPCs** (Remote Procedure Calls)

## 10. 远程过程调用 (Remote Procedure Calls, RPCs)

![[截屏2025-04-07_10.51.10.png]]

### 10.1 基本概念

- 在1970年代早期由Bruce Nelson引入，首次在PARC使用
- 减少调用者和被调用者之间的命运共享
- 由于通信延迟，RPC比本地调用耗时更长

### 10.2 RPC语义

- **At least once**：消息可能被重发多次，答案可能永远不会收到，适用于无副作用的操作
- **At most once**：消息最多被执行一次，发送者为接收响应设置超时，超时后向调用者传递错误代码，需要发送者保留时间历史，适用于有副作用的操作
- **Exactly once**：实现"at most once"语义并请求服务器确认

## 11. 分层 (Layering)

### 11.1 基本原则

- **Layering**需要模块化：每一层履行明确定义的功能
- 通信模式更具限制性，一层只与相邻层通信，这种限制降低了系统复杂性
- 严格执行的分层可能阻止优化，如无线应用中的跨层通信允许利用数据链路层MAC子层的信息

### 11.2 通信协议分层

- **Internet protocol stack**包括物理层、数据链路层、网络层、传输层和应用层
    - **Physical layer**适应多样化的物理通信通道
    - **Data link layer**解决通过通信通道直接连接的两个系统之间传输位的问题
    - **Network layer**处理从源到目的地的数据包转发
    - **Transport layer**保证从源到目的地的传输
    - **Application layer**数据只在应用上下文中有意义

## 12. 对等网络系统 (Peer-to-peer Systems, P2P)

### 12.1 基本特点

- P2P代表着对客户端-服务器模型的重大偏离
- 理想特性：
    - 需要最少的专用基础设施，参与系统贡献资源
    - 高度去中心化
    - 可扩展，单个节点不需要了解全局状态
    - 对故障和攻击有弹性，资源丰富支持高度复制
    - 单个节点不需要过多网络带宽
    - 动态和通常无结构的系统架构防止审查
- 不理想特性：
    - 去中心化提出P2P系统是否能有效管理和提供安全性的问题
    - 防审查使其成为非法活动的沃土

### 12.2 P2P系统中的资源共享

- 这种分布式计算模型促进对参与系统提供的存储和CPU周期的低成本访问
- 资源位于不同的管理域中
- P2P系统是自组织和去中心化的，而云中的服务器在单一管理域中并有中央管理
- 案例：**Napster**（音乐共享系统）和**SETI@home**（志愿者计算）

### 12.3 P2P系统的组织

- P2P系统构建在**overlay network**（叠加网络）之上，这是叠加在真实网络上的虚拟网络
- 两种类型的叠加网络：**unstructured**和**structured**
- 无结构叠加通常使用从几个引导节点开始的随机行走
- 结构化叠加中每个节点有唯一键，确定其在结构中的位置
- 结构化叠加网络使用**key-based routing (KBR)**
- 无结构叠加经常使用**epidemic algorithms**传播网络拓扑

### 12.4 P2P系统示例

- **Skype**：VoIP电话服务，近7亿注册用户
- 数据流应用，如**Cool Streaming**
- **BBC**的在线视频服务
- 内容分发网络如**CoDeeN**
- 基于**BOINC**平台的志愿计算应用

## 参考资料

- Marinescu, Dan C. Cloud Computing: Theory and Practice. Morgan Kaufmann, Third Edition