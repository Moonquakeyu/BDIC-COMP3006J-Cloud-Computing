  

## 1. Ray 简介与背景 (Introduction and Background)

### 1.1 AI 计算需求的快速增长

- AI 正在颠覆各个行业，从金融到医疗保健、交通和制造业
- **计算需求增长速度** (Computational demand growth) 极快：每18个月增长10倍
- 这种增长速度远超单节点处理器性能的提升，使得**分布式处理** (Distributed processing) 成为必然选择

### 1.2 Ray 的定义与价值

- **Ray** 是由 UC Berkeley 团队开发的统一分布式计算框架 (unified distributed framework)
- 专为支持新兴 AI 应用而设计，能够支持多种 AI 工作负载
- Ray 提供了统一接口，可以同时表达**任务并行** (task-parallel) 和**基于 Actor 的计算** (actor-based computations)
- 系统性能优越：能够扩展到每秒处理超过 **1.8 million** 个任务

## 2. AI 应用系统面临的挑战 (Challenges in AI Application Systems)

### 2.1 AI 应用的多阶段特性

典型 AI 应用涉及多个处理阶段:

- **预处理** (Preprocessing)：数据摄取、特征化等
- **批处理训练** (Batch Training)：模型训练
- **调优** (Tuning)：超参数优化
- **预测** (Prediction)：模型推理
- **服务** (Serving)：向用户提供预测结果

### 2.2 多系统集成的问题

使用多个不同系统处理各阶段带来的挑战:

1. **开发困难** (Hard to develop)：每个系统有自己的 API
2. **部署和管理复杂** (Hard to deploy and manage)：多系统环境
3. **语义不一致** (Inconsistent semantics)：各系统在容错和数据一致性方面有不同语义
4. **性能瓶颈** (Performance bottlenecks)：在阶段之间移动数据通常需要通过持久存储，速度慢

### 2.3 Ray 的解决方案

- 提供一个**统一计算框架** (unified compute framework) 支持所有工作负载
    
    ![[截屏2025-04-09_13.39.25.png]]
    
- 用不同的 Ray 库来扩展 ML 流水线的各个阶段，而非使用不同的系统
- 使用单一系统实现整个机器学习应用，避免拼接多个分布式系统

## 3. 并行与分布式计算基础 (Basics of Parallel and Distributed Computing)

### 3.1 计算模式对比

- **串行计算** (Serial computing)：单 CPU 顺序执行多个步骤
- **并行计算** (Parallel computing)：多个任务同时执行
- **分布式计算** (Distributed computing)：任务在多台机器上处理

### 3.2 Ray 集群架构 (Ray Cluster Architecture)

![[截屏2025-04-09_13.42.42.png]]

Ray 集群包含以下组件:

- **Head Node** (头节点)：包含驱动程序 (Driver) 和工作进程 (Worker)
- **Worker Nodes** (工作节点)：执行计算任务的节点
- **Raylet**：每个节点上的系统组件，包含：
    - **Scheduler** (调度器)
    - **Object Store** (对象存储)
- **Global Control Store (GCS)**：全局控制存储，管理集群状态

## 4. Ray 核心抽象 (Ray Core Abstractions)

### 4.1 三大核心抽象

Ray 提供三种核心抽象:

1. **Tasks** (任务)：
    
    - 无状态 Python 函数 (stateless Python functions)
    - 确定性 (Deterministic)
    - 引用透明 (Referential Transparency)
    - 易于测试 (Easy to Test)
    
    ```Python
    def add(a+b):
    	return a+b
    ```
    
    - 使用 `@ray.remote` 装饰器定义
2. **Actors** (角色)：
    - 有状态 Python 类 (stateful Python classes)
    - 具有内部状态 (Internal State)
    - 可变方法 (Mutating Methods)
    - 在方法调用之间维持状态 (Maintains State Across Method Calls)
    - 使用 `@ray.remote` 装饰器定义
3. **Objects** (对象)：
    - 可在 Ray 集群的不同组件之间共享和访问的数据

### Ray 的三种核心抽象详细解释

### 1. Tasks (任务)

**Tasks** 是 ==Ray 中最基本的无状态计算单元：==

- **无状态 (Stateless)** 意味着每个任务执行后不保留任何状态信息，每次执行都是全新的、独立的计算
- **确定性 (Deterministic)** 表示给定相同的输入，总是产生相同的输出
- **引用透明性 (Referential Transparency)** 指函数调用可以被其结果替换而不改变程序行为

举个简单例子：

```Python
# 定义一个普通 Python 函数
def add(a, b):
    return a + b

# 将其变成 Ray 远程任务
@ray.remote
def add_remote(a, b):
    return a + b

# 正常调用
result1 = add(10, 20)  # 直接返回 30

# Ray 远程调用
result2_future = add_remote.remote(10, 20)  # 立即返回一个 future 对象
result2 = ray.get(result2_future)  # 获取实际结果 30
```

**任务的关键特点**：

- 函数在远程工作进程上执行，而不是在当前进程
- 调用后立即返回一个 future 对象，让程序可以继续执行其他工作
- 可以用 `ray.get()` 获取结果，或将 future 传递给其他远程函数

  

### 2. Actors (角色)

**Actors** 是有==状态的计算单元==，基于 Python 类实现：

- **有状态 (Stateful)** 意味着 actor 可以在多次方法调用之间保持内部状态
- 适合需要维护状态的场景，如累加器、计数器或复杂模型

举个例子：

```Python
# 定义一个普通 Python 类
class Counter:
    def __init__(self):
        self.value = 0

    def increment(self):
        self.value += 1
        return self.value

    def get_value(self):
        return self.value

# 将其变成 Ray actor 类
@ray.remote
class CounterActor:
    def __init__(self):
        self.value = 0

    def increment(self):
        self.value += 1
        return self.value

    def get_value(self):
        return self.value

# 普通使用方式
counter = Counter()
counter.increment()  # 返回 1
counter.increment()  # 返回 2

# Ray actor 使用方式
counter_actor = CounterActor.remote()  # 创建 actor 实例
value1_future = counter_actor.increment.remote()  # 返回 future
value1 = ray.get(value1_future)  # 获取值 1
value2_future = counter_actor.increment.remote()  # 返回 future
value2 = ray.get(value2_future)  # 获取值 2
```

**Actor 的关键特点**：

- Actor 是==一个专用进程，维护自己的内部状态==
- 方法调用是序列化的（按顺序执行），确保状态一致性
- Actor 可以在不同节点上运行，用于分布式有状态计算

### 3. Objects (对象)

**Objects** 是 ==Ray 中的数据容器==：

- 表示任务或 actor 方法产生的数据
- 存储在分布式对象存储中，可以在集群的不同组件之间共享
- 由 Object ID (对象 ID) 标识，类似于指向数据的引用或指针

在实际使用中：

```Python
# 远程任务返回的是 Object ID
object_id = add_remote.remote(10, 20)

# ray.put() 可以将 Python 对象放入对象存储，返回 Object ID
data_id = ray.put([1, 2, 3, 4])

# 可以将 Object ID 传递给其他任务
result_id = process_data.remote(data_id)

# 使用 ray.get() 获取对象的实际内容
actual_data = ray.get(data_id)  # 返回 [1, 2, 3, 4]
```

**Objects 的关键特点**：

- 提供分布式数据共享机制，避免数据复制
- 自动处理数据依赖关系，确保任务在需要的数据可用时执行
- 支持数据局部性优化，减少网络传输

### 这三种抽象如何协同工作？

想象一个简单的机器学习训练场景：

1. **Objects** 存储训练数据和模型参数
2. **Tasks** 用于并行处理数据，如数据预处理和批量梯度计算
3. **Actors** 用于维护模型状态并执行训练循环

```Python
# 简化示例
@ray.remote
def preprocess_batch(data_batch):
    # 无状态数据处理
    return processed_data

@ray.remote
class ModelTrainer:
    def __init__(self, model_config):
        self.model = initialize_model(model_config)
        self.optimizer = initialize_optimizer()

    def train_step(self, batch_data):
        # 更新模型状态
        loss = compute_loss(self.model, batch_data)
        self.optimizer.step(loss)
        return loss.item()

    def get_model_params(self):
        return self.model.parameters()
```

通过这种组合，Ray 能够有效地支持复杂的分布式计算流程，使 AI 应用能够扩展到多节点集群上运行。

### 4.2 Actor 使用模式

Ray 原生库中使用的 Actor 模式:

- **Tree of Actors** (Actor 树)：用于训练多个模型，使用不同的机器学习算法
- **Same Data Different Function/Model (SDDF)**：同时使用相同数据集训练不同模型

## 5. 强化学习工作负载 (Reinforcement Learning Workloads)

### 5.1 机器学习类型简介

三种主要的机器学习类型:

- **监督学习** (Supervised Learning)：使用带标签的数据
- **无监督学习** (Unsupervised Learning)：使用无标签数据
- **强化学习** (Reinforcement Learning)：通过与环境交互和奖励机制学习

### 5.2 强化学习系统示例

强化学习系统的工作流程:

![[截屏2025-04-09_13.56.29.png]]

- **Agent** (代理) 与 **Environment** (环境) 不断交互
- Agent 的目标是学习一个能够最大化奖励的 **Policy** (策略)
- Policy 将环境状态映射到动作选择
- 包括 **Policy Evaluation** (策略评估) 和 **Policy Improvement** (策略改进) 两个关键步骤

### 5.3 强化学习的工作负载特点

强化学习应用的特殊工作负载:

- **Training** (训练)：通常使用分布式随机梯度下降 (SGD) 更新策略
- **Serving** (服务)：使用训练好的策略根据环境当前状态呈现动作
- **Simulation** (模拟)：使用模拟评估策略，模拟复杂度变化很大
- 这三种工作负载在强化学习应用中紧密耦合，有严格的延迟要求

## 6. Ray 编程模型与计算模型 (Ray Programming and Computational Model)

### 6.1 Tasks 详解

Tasks 的特点与使用:

- 代表在无状态工作进程上执行的远程函数
- 调用远程函数时，立即返回表示任务结果的 **future**
- 可以使用 `ray.get()` 获取 **futures 的结果**
- Futures 可以作为参数传递给其他远程函数
- 允许用户表达并行性同时捕获数据依赖关系

### 6.2 Actors 详解

Actors 的特点与使用:

- 代表有状态的计算
- 每个 actor 暴露可远程调用的方法，这些方法按顺序执行
- 方法执行类似于任务，但在有状态工作进程上执行
- Actor 句柄可以传递给其他 actors 或 tasks

### 6.3 Tasks 与 Actors 的权衡

两种模型的比较:

- **Tasks (无状态)**:
    - 细粒度负载均衡 (Fine-grained load balancing)
    - 支持对象局部性 (Support for object locality)
    - 小更新的高开销 (High overhead for small updates)
    - 高效故障处理 (Efficient failure handling)
- **Actors (有状态)**:
    - 粗粒度负载均衡 (Coarse-grained load balancing)
    - 局部性支持较差 (Poor locality support)
    - 小更新的低开销 (Low overhead for small updates)
    - 检查点带来的开销 (Overhead from checkpointing)

  

- **Tasks: 细粒度负载均衡 (Fine-grained load balancing)**
    - 由于任务是无状态的，每个任务可以被独立调度到任何可用工作节点
    - 系统可以动态分配任务，实现更精细的工作负载分布
    - 允许更好的资源利用率，特别是在工作负载不均匀的情况下
- **Actors: 粗粒度负载均衡 (Coarse-grained load balancing)**
    - Actor 在创建后固定在特定节点上
    - 所有对该 Actor 的方法调用都必须在该节点上执行
    - 负载均衡只发生在 Actor 创建时，无法动态调整
- **Tasks: 支持对象局部性 (Support for object locality)**
    - 任务可以被调度到输入数据所在的节点上执行
    - Ray 调度器会考虑数据位置，减少数据传输
    - 这种数据局部性优化可以显著减少网络开销
- **Actors: 局部性支持较差 (Poor locality support)**
    - Actor 位置固定，不能移动到数据所在位置
    - 如果 Actor 需要处理分布在不同节点的数据，数据必须移动到 Actor 所在节点
    - 这可能导致更多的网络传输开销

### 6.4 Ray 计算模型

Ray 的动态任务图计算模型:

- 当输入可用时，系统自动触发远程函数和 actor 方法的执行

- 计算图中的节点类型:
    1. 数据对象 (Data objects)
    2. 远程函数调用或任务 (Remote function invocations or tasks)
    3. Actor 方法调用 (Actor method invocations)

![[截屏2025-04-09_14.07.25.png]]

- 边的类型:
    1. 数据边 (Data edges)：捕获数据对象和任务之间的依赖关系
    2. 控制边 (Control edges)：捕获嵌套远程函数产生的计算依赖关系
    3. 状态边 (Stateful edge)：捕获同一 actor 上连续方法调用之间的状态依赖关系

## 7. Ray 架构 (Ray Architecture)

### 7.1 应用层 (Application Layer)

应用层包含三种类型的进程:

- **Driver** (驱动程序)：执行用户程序的进程
- **Worker** (工作进程)：执行任务的无状态进程，由系统层自动启动和分配
- **Actor** (角色)：执行其公开方法的有状态进程，由 worker 或 driver 显式实例化

### 7.2 系统层 (System Layer)

系统层由三个主要组件组成:

1. **Global Control Store (GCS)**
    - 维护系统的所有控制状态
    - 核心是具有发布-订阅功能的键值存储
2. **Distributed Scheduler** (分布式调度器)
    - 两级层次结构：全局调度器和每节点本地调度器
    - 本地调度器优先在本地调度任务
    - 如果节点过载或无法满足任务要求，则转发到全局调度器
    - 采用自下而上的调度策略 (bottom-up scheduler)
3. **Distributed Object Store** (分布式对象存储)
    - 内存中的分布式存储系统，存储每个任务的输入和输出
    - 每个节点上的对象存储通过共享内存实现
    - 如果任务输入不在本地，将在执行前复制到本地对象存储
    - 复制消除了热数据对象可能造成的瓶颈，最小化任务执行时间

### 7.3 系统架构图解

Ray 系统层包含多个节点:

- 节点间通过 GCS 协调
- 本地调度器负责任务分配
- 全局调度器处理负载均衡

## 8. 总结 (Summary)

Ray 是为新兴 AI 应用设计的统一分布式框架，具有以下关键特点：

1. 提供 Tasks 和 Actors 两种核心抽象，适应不同计算需求
2. 统一接口简化开发，避免多系统集成的复杂性
3. 分层架构确保高性能和容错性
4. 特别适合强化学习等要求紧密耦合计算的 AI 应用场景
5. 可水平扩展，性能优越，支持每秒百万级任务处理

Ray 通过单一系统支持 AI 应用的各个阶段，为 AI 计算提供了灵活、高效的分布式计算解决方案。