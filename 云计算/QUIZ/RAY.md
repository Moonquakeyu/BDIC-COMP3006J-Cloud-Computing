## 1. 简介与动机 (Introduction and Motivation)

### 1.1 背景 (Background)

- 下一代 AI 应用将持续与环境交互并从这些交互中学习
- 这些应用提出了新的系统要求，包括性能和灵活性方面
- 现有的数据处理框架（包括批处理、流处理和图处理系统）无法满足这些新兴 AI 应用的需求

### 1.2 强化学习的特殊需求 (Special Requirements for RL)

- **强化学习 (Reinforcement Learning, RL)** 是一种 AI 技术，其中 agent 通过与环境交互学习最优策略
- RL 系统包含三个关键组件：
    1. **Simulation (模拟)**: 探索可能的动作序列
    2. **Training (训练)**: 优化策略参数
    3. **Serving (服务)**: 在实时控制场景中部署策略
- 现有系统难以同时支持这三个紧密耦合的组件

### 1.3 Ray 的价值主张 (Value Proposition)

- Ray 是一个通用的分布式框架，能够支持新兴 AI 应用的需求
- 统一实现了 simulation、training 和 serving 功能
- 提供了可扩展的架构，能处理超过每秒 180 万个任务
- 在多个具有挑战性的强化学习应用中优于现有专用系统

## 2. 系统设计要求 (System Design Requirements)

### 2.1 关键技术要求 (Key Technical Requirements)

1. **Fine-grained computations (细粒度计算)**:
    - 任务可能持续从几毫秒到几小时不等
    - 系统必须高效处理数百万个短小任务
2. **Heterogeneous computations (异构计算)**:
    - 处理时间和资源需求各不相同的任务
    - 需要支持 CPU 和 GPU 等不同硬件资源
3. **Dynamic execution (动态执行)**:
    - 工作负载可能基于模拟结果或环境反馈而变化
    - 要求系统能动态适应变化的计算图

### 2.2 与现有系统的比较 (Comparison with Existing Systems)

- **批处理系统** (MapReduce, Spark): 不支持细粒度模拟或策略服务
- **任务并行系统** (CIEL, Dask): 对分布式训练和服务支持有限
- **深度学习框架** (TensorFlow, MXNet): 不能自然支持模拟和服务
- **模型服务系统** (TensorFlow Serving): 不支持训练或模拟

## 3. 编程和计算模型 (Programming and Computation Model)

### 3.1 统一编程接口 (Unified Programming Interface)

Ray 提供两种核心抽象:

1. **Tasks (任务)**: 无状态远程函数执行
    - 通过 `@ray.remote` 装饰器定义
    - 返回 future 对象，可以传递给其他任务
    - 无状态性使得故障恢复更简单（通过重新执行函数）
2. **Actors (参与者)**: 有状态计算单元
    - 通过 `@ray.remote` 装饰类定义
    - 维护跨方法调用的状态
    - 方法调用按顺序执行，依赖之前的状态

### 3.2 核心 API 功能 (Core API Functions)

- `f.remote(args)`: 远程执行函数，返回 future
- `ray.get(futures)`: 阻塞获取 future 的值
- `ray.wait(futures, k, timeout)`: 等待 k 个任务完成或超时
- `Class.remote(args)`: 实例化远程 actor
- `actor.method.remote(args)`: 调用远程 actor 方法

### 3.3 Tasks 与 Actors 对比 (Tasks vs Actors Comparison)

|   |   |
|---|---|
|Tasks (无状态)|Actors (有状态)|
|细粒度负载均衡|粗粒度负载均衡|
|支持对象局部性|对局部性支持有限|
|小更新开销高|小更新开销低|
|高效故障处理|需要检查点开销|

### 3.4 计算图模型 (Computation Graph Model)

- Ray 采用**动态任务图计算模型**
- 系统自动触发执行远程函数和 actor 方法
- 图中节点包括数据对象和任务调用
- 边包括数据依赖和控制依赖
- 动态图可在执行过程中根据任务结果修改

## 4. 系统架构 (System Architecture)

### 4.1 架构概览 (Architecture Overview)

Ray 架构分为两层:

1. **应用层 (Application Layer)**: 实现 API 和计算模型
2. **系统层 (System Layer)**: 实现任务调度和数据管理

系统层包含三个主要组件:

- **全局控制存储 (Global Control Store, GCS)**
- **分布式调度器 (Distributed Scheduler)**
- **分布式对象存储 (Distributed Object Store)**

### 4.2 全局控制存储 (Global Control Store, GCS)

- 维护系统的全部控制状态
- 本质上是具有发布-订阅功能的键值存储
- 使用分片实现扩展性，每个分片使用链式复制提供容错能力
- 存储系统元数据，包括对象表、任务表和函数表
- 通过中央化控制状态简化了 Ray 的开发和调试

### 4.3 分布式调度器 (Distributed Scheduler)

Ray 实现了分层调度架构:

1. **全局调度器 (Global Scheduler)**:
    - 管理跨节点任务分配
    - 负责实现全局负载均衡
    - 可水平扩展以处理大量任务
2. **本地调度器 (Local Scheduler)**:
    - 运行在每个节点上
    - 处理本地任务分配
    - 缓存本地对象元数据
    - 维护等待输入的任务队列和可调度任务

### 4.4 分布式对象存储 (Distributed Object Store)

- 通过共享内存实现，允许同一节点上的任务零拷贝数据共享
- 所有对象都是不可变的，简化了一致性和容错支持
- 维护对象存储在内存中，必要时通过 LRU 策略驱逐到磁盘
- 使用 Apache Arrow 作为数据格式，提供高效序列化/反序列化

### 4.5 容错机制 (Fault Tolerance)

Ray 提供两种容错机制:

1. **基于谱系的容错 (Lineage-based)**:
    - 用于任务和对象
    - 通过重新执行生成对象的任务来恢复丢失对象
2. **基于复制的容错 (Replication-based)**:
    - 用于元数据存储
    - GCS 表按对象和任务 ID 分片
    - 每个分片使用链式复制提供容错能力

## 5. 系统性能与评估 (System Performance and Evaluation)

### 5.1 微基准测试 (Microbenchmarks)

- **任务吞吐量**: 可扩展至每秒 180 万个任务
- **Locality-aware 任务放置**: 对于大数据输入比非局部性感知放置提高 1-2 个数量级的性能
- **Allreduce 操作**: 在 16 节点上处理 100MB 数据时达到接近理论最优性能

### 5.2 与专用系统比较 (Comparison with Specialized Systems)

### 5.2.1 训练性能 (Training Performance)

- Ray 在大规模分布式训练中表现接近或超过专用系统如 Horovod
- 优化包括梯度计算、传输和累加的流水线化
- 使用自定义 TensorFlow 算子将张量直接写入 Ray 的对象存储

### 5.2.2 服务性能 (Serving Performance)

- 与专用服务系统 Clipper 相比，Ray 提供更高吞吐量:
    - 小型全连接网络: Ray 每秒 6200 状态 vs Clipper 4400 状态
    - 较大输入情况: Ray 每秒 6900 状态 vs Clipper 290 状态
- 性能提升归功于低开销序列化和共享内存抽象

### 5.2.3 模拟性能 (Simulation Performance)

- Ray 实现了高效的负载均衡和并行处理
- 通过分层聚合技术处理大规模模拟
- 有效处理异构任务持续时间和资源需求

### 5.3 强化学习应用案例 (RL Application Case Studies)

### 5.3.1 Evolution Strategies (ES)

- Ray 实现可扩展到 8192 个核心
- 使用 actor 聚合树达到中位数 3.7 分钟完成时间，比最佳发布结果快两倍多
- 初始并行化仅需修改 7 行代码，而参考实现包含数百行专用通信协议代码

### 5.3.2 Proximal Policy Optimization (PPO)

- Ray 实现显著优于基准实现
- 通过简单 GPU 训练和 CPU 模拟任务组合实现高效混合架构
- 易于表达复杂优化，如并行策略优化和权重共享

## 6. Ray 与相关工作比较 (Ray vs Related Work)

### 6.1 动态任务图系统 (Dynamic Task Graph Systems)

- Ray 与 CIEL 和 Dask 相似，都支持动态任务图和嵌套任务
- Ray 的主要优势:
    1. 扩展任务模型加入 actor 抽象，支持高效有状态计算
    2. 采用完全分布式和解耦的控制平面和调度器

### 6.2 数据流系统 (Dataflow Systems)

- 与 MapReduce、Spark 和 Dryad 等系统相比:
    - 这些系统计算模型对细粒度动态模拟工作负载过于限制
    - 缺乏 actor 抽象和分布式可扩展控制平面

### 6.3 机器学习框架 (ML Frameworks)

- TensorFlow 和 MXNet 专注于深度学习工作负载:
    - 对静态 DAG 线性代数操作优化良好
    - 对将训练与模拟和服务紧密耦合的通用计算支持有限

### 6.4 Actor 系统 (Actor Systems)

- Orleans 和 Akka 是面向开发高可用并发分布式系统的 actor 框架:
    - 对数据丢失恢复支持较少
    - 缺少 Ray 提供的透明容错和精确一次语义

## 7. 经验教训与限制 (Lessons Learned and Limitations)

### 7.1 关键经验 (Key Lessons)

- **中央控制状态**的价值:
    - GCS 显著简化了 Ray 的开发和调试
    - 使全局调度器能够通过添加更多副本来扩展
    - 中央化控制状态将成为未来分布式系统的关键设计组件

### 7.2 系统限制 (Limitations)

- 工作负载通用性使特定优化变得困难
- 必须在不完全了解计算图的情况下进行调度决策
- 为每个任务存储谱系需要实现垃圾回收策略以限制 GCS 存储成本

### 7.3 容错重要性 (Importance of Fault Tolerance)

- 容错对 AI 应用至关重要的原因:
    1. 忽略故障使应用程序更容易编写和推理
    2. 通过确定性重放大大简化调试
    3. 允许使用 AWS spot 实例等便宜资源，节省成本

## 8. 结论与未来方向 (Conclusion and Future Directions)

### 8.1 总结 (Summary)

- Ray 统一了任务并行和 actor 编程模型于单一动态任务图中
- 采用由全局控制存储和自下而上分布式调度器实现的可扩展架构
- 实现了线性扩展至每秒 180 万个任务，同时提供透明容错
- 在多个现代强化学习工作负载上取得显著性能改进

### 8.2 Ray 生态系统 (Ray Ecosystem)

- Ray 成为分布式 AI 应用的基础框架
- 提供与 Python 环境的完全集成，安装简单 (`pip install ray`)
- 开源项目持续发展，支持更广泛的 AI 和分析工作负载

### 8.3 影响与贡献 (Impact and Contributions)

1. 首个统一训练、模拟和服务的分布式框架
2. 在动态任务执行引擎之上统一 actor 和任务并行抽象
3. 采用中央控制状态存储和自下而上调度策略实现可扩展性和容错
4. 为新兴 AI 应用提供灵活性、性能和易用性的强大组合

## 专有名词汇总 (Terminology)

- **Ray**: 为新兴 AI 应用设计的分布式框架
- **Reinforcement Learning (RL)**: 通过环境交互学习的 AI 技术
- **Task**: Ray 中的无状态远程函数执行
- **Actor**: Ray 中的有状态计算单元
- **Global Control Store (GCS)**: Ray 的中央化控制状态存储
- **Distributed Scheduler**: Ray 的分层任务调度系统
- **Object Store**: 管理分布式内存中对象的组件
- **Lineage**: 对象创建历史，用于容错
- **Future**: 表示异步计算结果的对象
- **Dynamic Task Graph**: 动态演化的计算依赖图
- **Task-parallel**: 基于无状态任务的并行计算模型
- **Actor-based**: 基于有状态 actor 的计算模型

Ray 通过统一编程接口、高性能分布式执行引擎和创新的系统架构，为强化学习等新兴 AI 应用提供了强大的框架支持，代表了分布式系统向支持更复杂 AI 工作负载演进的重要里程碑。