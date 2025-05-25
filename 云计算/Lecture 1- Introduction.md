# Lecture 1: Introduction

## 1. Cloud Computing 概念背景

### 1.1 数据世界的现状

- **Big data** 被定义为可以被捕获、传输、聚合、存储和分析的大量数据池
- 数据持续增长：2010年中期，信息宇宙承载了1.2 ZB的数据，到2020年，数据量增加了近44倍
- Applications 正变得日益**data-intensive**

### 1.2 数据处理需求

我们希望能够无缝地（**seamlessly**）：

- **Store** 数据
- **Access** 数据
- **Encrypt** 数据
- **Share** 数据
- **Process** 数据
- 以及更多功能

### 1.3A 多样化设备和接口

我们希望随时随地通过各种设备访问、共享和处理数据：

- **Desktops**
- **Mobile Devices**
- **Consumer Electronics**
- 甚至家用电器（**appliances**）

### 1.3B 其他日常需求

用户需要解决的问题包括：

- 处理**documents**
- 创建、访问、存储和共享**media**
- 获取**news and information**
- **Navigate**
- 与朋友和家人**communicate**
- 在**smart home**中生活
- 等等

## 2. 从**Product**到**Service**的转变

### 2.1 数据管理方式的转变

数据管理方法的演变：

- **Manage it ourselves**：个人化但耗时
- **Managed by someone else**：可以免费或通过**subscription**获得服务

### 2.2 历史上的类似转变

**Innovation → Product → Service** 的转变模式

### 水资源利用的演变：

1. **Generate your own utility**
2. **Buy it as a product and manage it**
3. **Get a continuous supply through a dedicated connection**

### 电力的转变：

1. **Innovation**: 破坏性新技术
2. **Product**: 购买和维护技术
3. **Service**: **Electric Grid**，只为使用的电力付费

### 银行业务的演变：

1. **No Banks**（自己保管钱财）
2. **Traditional Banking**（将钱存入银行）
3. **Banking Instruments**（支票/信用卡）
4. **Internet Banking**（更多服务）

## 3. Cloud Computing 的定义

### 3.1 核心定义

- "**Cloud Computing is the transformation of IT from a product to a service**"
- **Innovation of IT** → **IT Products** → **Cloud Computing**（按需付费的IT服务）

### 3.2 NIST官方定义

美国国家标准与技术研究院(NIST)定义云计算为：

"**Cloud computing** is a model for enabling **ubiquitous**, **convenient**, **on-demand network access** (无处不在，方便，按需应变) to a shared pool of configurable computing resources (e.g., networks, servers, storage, applications, and services) that can be **rapidly provisioned and released** with minimal management effort or service provider interaction."

### 3.3 云计算的实用性标准

判断服务是否以云计算方式提供的三个标准：

1. 服务可通过**Web browser**(非专有)或**Web services API**访问
2. 不需要前期**capital expenditure**即可开始使用
3. 只为使用的资源付费（**pay only for what you use**）

### 3.4 作为分布式系统的云

云是一种**parallel and distributed system**，由相互连接的**virtualized computers**组成，这些计算机根据服务提供商和消费者之间通过协商建立的**service-level agreements**，被动态供应并呈现为一个或多个统一的计算资源。

## 4. Cloud Computing 的部署模型

云计算有多种主要部署模型，具体包括：

- **Public Cloud**
- **Private Cloud**
- **Community Cloud**
- **Hybrid Cloud**

## 5. Cloud Computing 参考模型

### 5.1 服务层次结构

云计算参考模型包括不同的服务层次：

- **Software as a Service (SaaS)**
- **Platform as a Service (PaaS)**
- **Infrastructure as a Service (IaaS)**

### 5.2 责任边界

云用户和云服务提供商之间的**limits of responsibility**根据服务类型而有所不同。

## 6. Cloud Computing 的优势

### 6.1 经济模型优势

**Pay-as-You-Go economic model**：

- 减少**capital expenditure**
- 没有**upfront cost**
- 缩短**Time to Market**

### 6.2 管理优势

**Simplified IT management**：

- 只需要**access to the internet**
- 提供商负责管理细节

### 6.3 可扩展性优势

**Scale quickly and effortlessly**：

- 资源可根据需求**rented and released**
- **Software Controlled**
- **Instant scalability**

### 6.4 灵活性优势

**Flexible options**：

- 可配置**software packages**, **instance types** 和**operating systems**
- 任何**software platform**
- 从任何连接到互联网的机器访问

### 6.5 资源利用优势

**Resource Utilization is improved**：

- 通过**sharing and consolidation**减少闲置资源
- 更好地利用**CPU/Storage and Bandwidth**

### 6.6 环保优势

**Carbon Footprint decreased**：

- 资源共享意味着更少的服务器、更少的能源消耗和更少的排放

## 7. Cloud Computing 经济学

### 7.1 有利于公用计算的三个案例

1. 服务需求随时间变化时（**demand varies with time**）
2. 需求事先未知时（**demand is unknown in advance**）
3. 执行批量分析的组织可以利用云计算的**"cost associativity"**更快完成计算

### 7.2 适合云计算的应用场景

### 7.2.1 **High Growth Applications**

- 适合快速增长的初创企业
- 案例：**Animoto**能够在3天内将服务器从50台扩展到3500台，然后再减少

### 7.2.2 **Aperiodic Bursting Applications**

- 适合流量呈现不规律峰值的网站
- 案例：**9/11**事件导致新闻网站宕机、**情人节**花店网站高峰、**Superbowl**促销期间

### 7.2.3 **On-Off Applications**

- 适合研究人员使用数千台计算机进行大规模科学模拟
- 案例：现代**Drug Discovery**需要数据密集型模拟和测试

### 7.2.4 **Periodic Applications**

- 适合计算需求随时间变化的场景
- 案例：**Stock Market Analysis**（白天挖掘市场数据，晚上处理和分析）

## 8. 云计算的资源配置挑战

### 8.1 为峰值负载配置

在没有**elasticity**的情况下，即使能够正确预测**peak load**，非峰值时段仍然会浪费资源

![[截屏2025-04-07_09.47.29.png]]

### 8.2 配置不足的风险

- 放弃了未服务用户的潜在**revenue**
    
    ![[截屏2025-04-07_09.47.47.png]]
    
- 一些用户在体验糟糕的服务后会永久离开，导致永久性**revenue loss**
    
    ![[截屏2025-04-07_09.48.39.png]]
    

### 8.3 **Usage-based pricing** 与租赁的区别

- **Renting a resource**：支付协商成本以在一段时间内拥有资源，无论是否使用
- **Pay-as-you-go**：计量使用情况并根据实际使用情况收费，与使用的时间段无关

## 9. 总结

**Cloud Computing** 代表了IT从产品到服务的根本性转变，提供了**on-demand**、**scalable**、**cost-effective**的计算资源访问方式。云计算的**deployment models**和**service types**多样，可以满足不同组织和用户的需求。云计算的优势包括**economic model**、**management simplification**、**scalability**、**flexibility**、**resource optimization**和**environmental benefits**等方面，特别适合需求波动、快速增长和计算密集型的应用场景。