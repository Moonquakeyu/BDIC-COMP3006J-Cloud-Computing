  

## 1. Pod 的组织 (Organizing Pods)

### 1.1 扁平的跨 Pod 网络

- Kubernetes 集群中的所有 Pod 都位于单一的、共享的**network-address space**
- 每个 Pod 都可以通过其 IP 地址访问任何其他 Pod
- Pod 之间不存在**NAT** (Network Address Translation)
- 当两个 Pod 相互发送网络数据包时，它们会在数据包中看到对方的实际 IP 地址
    
    ![[截屏2025-04-08_09.49.31.png]]
    

### 1.2 跨 Pod 组织容器的原则

- 不应该将所有内容都塞进单个 Pod，而应该将应用程序组织成多个 Pod
- 每个 Pod 只应包含紧密相关的组件或进程
- 多层应用程序（如前端和后端）应该分成多个 Pod 而不是单个 Pod

### 1.3 将多层应用拆分为多个 Pod 的优势

- **资源利用率提高**：如果前端和后端在同一个 Pod 中，它们将始终在同一台机器上运行
- 在双节点集群中，如果使用单一 Pod，将只能利用一个工作节点的计算资源
- 将 Pod 分成两个可以让 Kubernetes 将前端调度到一个节点，后端调度到另一个节点
- 这将提高基础设施的利用率

### 1.4 拆分 Pod 实现独立扩展

- Pod 是**basic unit of scaling**（基本缩放单位）
- Kubernetes 不能水平缩放单个容器，而是缩放整个 Pod
- 如果 Pod 由前端和后端容器组成，当扩展到两个实例时，会得到两个前端容器和两个后端容器

### 1.5 何时在 Pod 中使用多个容器

- 主要原因是当应用程序由一个主进程和一个或多个补充进程组成时
    
    ![[截屏2025-04-08_10.00.31.png]]
    
- 确定是否使用多容器 Pod 的问题：
    1. 它们是否需要一起运行，或者可以在不同主机上运行？
    2. 它们是代表一个整体，还是独立的组件？
    3. 它们必须一起缩放还是单独缩放？

### 1.6 容器与 Pod 最佳实践

![[截屏2025-04-08_10.12.38.png]]

- 一个容器不应该运行多个进程
- 如果容器不需要在同一台机器上运行，则 Pod 不应该包含多个容器

## 2. Pod 描述符 (Pod Descriptor)

### 2.1 使用 YAML 或 JSON 描述符创建 Pod

- Pod 和其他 Kubernetes 资源通常通过向 Kubernetes REST API 端点发送 JSON 或 YAML **manifest**来创建
- 使用 YAML 文件定义 Kubernetes 对象的优势：
    1. 允许配置更多属性
    2. 可以将文件存储在版本控制系统中

### 2.2 Pod 定义的主要部分

Pod 定义由几个部分组成：

1. Kubernetes API 版本和资源类型
2. **Metadata**：包括 Pod 的名称、命名空间、标签和其他信息
3. **Spec**：包含 Pod 内容的实际描述，如 Pod 的容器、卷和其他数据
4. **Status**：包含有关正在运行的 Pod 的当前信息，如 Pod 处于什么状态、每个容器的描述和状态、Pod 的内部 IP 等
    
    ![[截屏2025-04-08_10.16.51.png]]
    

## 3. 使用标签组织 Pod (Organizing Pods with Labels)

### 3.1 标签的重要性

- 随着 Pod 数量增加，需要将它们分类成子集
- 标签是组织 Pod 和所有其他 Kubernetes 对象的强大功能
- **Label**是附加到资源的任意**key-value pair**
- 可以使用**label selectors**选择具有特定标签的资源
- 一个资源可以有多个标签，只要这些标签的键在该资源中是唯一的

### 3.2 使用标签和选择器约束 Pod 调度

- 默认情况下，Pod 在工作节点之间随机调度
- 每个 Pod 获取其请求的确切计算资源（CPU、内存等）
- 某些情况下，可能希望对 Pod 的调度位置有一定控制：
    - 例如，当硬件基础设施不均质时
    - 如果部分工作节点有传统硬盘，而其他节点有 SSD
    - 或需要将执行 GPU 密集型计算的 Pod 调度到提供所需 GPU 加速的节点上
- 可以通过**node labels**和**node label selectors**描述节点需求，让 Kubernetes 选择匹配的节点

## 4. 保持 Pod 健康 (Keeping Pods Healthy)

### 4.1 Pod 健康的重要性

- Pod 是 Kubernetes 中的基本部署单元
- 希望部署能够自动保持运行和健康，无需手动干预
- 将容器列表提供给 Kubernetes，让它通过创建 Pod 资源并在集群中某处运行这些容器
- 关键问题：
    1. 如果其中一个容器死亡怎么办？
    2. 如果 Pod 的所有容器都死亡怎么办？

### 4.2 自动重启机制

- 一旦 Pod 被调度到节点上，该节点上的**Kubelet**将运行其容器，并在 Pod 存在期间保持容器运行
- 如果容器的主进程崩溃，**Kubelet**将重新启动容器
- 如果应用程序有偶尔崩溃的错误，Kubernetes 将自动重启它
- 即使在应用程序本身不做任何特殊处理的情况下，在 Kubernetes 中运行应用程序也自动具备了自我修复能力

## 5. 复制控制器 (ReplicationControllers)

### 5.1 ReplicationController 简介

- **ReplicationController**是确保其 Pod 始终运行的 Kubernetes 资源
- 如果 Pod 由于任何原因消失（如节点从集群中消失或 Pod 被从节点中驱逐），**ReplicationController**会注意到缺少的 Pod 并创建替代 Pod
    
    ![[截屏2025-04-08_20.22.33.png]]
    

### 5.2 受管 Pod 与非受管 Pod 的区别

- 直接创建的 Pod（Pod A）是非受管 Pod，而由 ReplicationController 管理的 Pod B 受到监管
- 当节点故障后，ReplicationController 会创建一个新的 Pod（Pod B2）来替换缺少的 Pod B
- 而 Pod A 则完全丢失，不会被重新创建
- ReplicationController 可以管理单个 Pod，但通常用于创建和管理 Pod 的多个副本

### 5.3 ReplicationController 的运行机制

- ReplicationController 持续监控正在运行的 Pod 列表
- 确保"类型"的 Pod 的实际数量始终与期望数量匹配
- 如果此类 Pod 运行太少，它会从 Pod 模板创建新的副本
- 如果此类 Pod 运行太多，它会删除多余的副本
- 可能出现超过期望数量副本的原因：
    1. 有人手动创建了相同类型的 Pod
    2. 有人更改了现有 Pod 的"类型"
    3. 有人减少了期望的 Pod 数量等

### 5.4 控制器的协调循环

ReplicationController 的工作是确保 Pod 的确切数量始终与其标签选择器匹配：

1. **查找**与标签选择器匹配的 Pod
2. **比较**匹配的与期望的 Pod 数量
3. 如果太少，从当前 Pod 模板**创建**额外的 Pod
4. 如果太多，**删除**多余的 Pod
5. 如果数量刚好，则不做任何操作

### 5.5 ReplicationController 的三个基本部分

1. **Label selector**：确定哪些 Pod 在 ReplicationController 的范围内
2. **Replica count**：指定应该运行的 Pod 的期望数量
3. **Pod template**：用于创建新的 Pod 副本

### 5.6 使用 ReplicationController 的好处

- 确保 Pod（或多个 Pod 副本）始终在运行，当现有 Pod 消失时启动新的 Pod
- 当集群节点故障时，为在故障节点上运行的所有 Pod 创建替代副本
- 实现 Pod 的水平扩展 —— 手动和自动
- 注意：Pod 实例永远不会被重新定位到另一个节点，ReplicationController 会创建一个全新的 Pod 实例，与它替换的实例没有关系

### 5.7 创建 ReplicationController

- 与 Pod 和其他 Kubernetes 资源一样，通过向 Kubernetes API 服务器发送 JSON 或 YAML 描述符来创建 ReplicationController
- 代码示例：

```Shell
apiVersion: v1
kind: ReplicationController
metadata:
  name: kubia
spec:
  replicas: 3
  selector:
    app: kubia
  template:
    metadata:
      labels:
        app: kubia
    spec:
      containers:
      - name: kubia
        image: luksa/kubia
        ports:
        - containerPort: 8080
```

## 6. 副本集 (ReplicaSets)

### 6.1 ReplicaSet 简介

- 最初，ReplicationController 是唯一用于复制 Pod 并在节点故障时重新调度它们的 Kubernetes 组件
- 后来引入了一种名为 ReplicaSet 的类似资源
- 它是 ReplicationController 的新一代，并完全取代它（ReplicationController 最终将被弃用）

### 6.2 ReplicaSet 与 ReplicationController 的比较

- ReplicationController 只支持基于相等的选择器
- ReplicaSet 支持新的基于集合的选择器
- ReplicaSet 的行为与 ReplicationController 完全相同，但具有更具表现力的 Pod 选择器
- ReplicationController 的标签选择器只允许匹配包含特定标签的 Pod
- ReplicaSet 的选择器还允许<font color="#ff0000">匹配缺少特定标签的 Pod 或包含特定标签键的 Pod</font>，无论其值如何
- 单个 ReplicationController 不能同时匹配带有标签 env=production 和带有标签 env=devel 的 Pod，但单个 ReplicaSet 可以匹配这两组 Pod
- ReplicaSet 可以基于标签键的存在匹配 Pod，而不考虑其值（例如 env=* ）

### **ReplicationController 的选择器（基于相等的选择器）**

ReplicationController 只能使用最基本的标签匹配方式，就像是"完全等于"的关系：

- 它只能选择**完全匹配**特定标签键值对的 Pod
- 例如：它只能选择 `env=production` 的 Pod，或者只能选择 `env=devel` 的 Pod，但不能同时选择这两种 Pod11
- 这就像是说："我只要标签 'env' 值正好等于 'production' 的 Pod"

### **ReplicaSet 的选择器（基于集合的选择器）**

ReplicaSet 提供了更灵活的标签匹配方式，它可以：

1. **同时匹配多种标签值**：
    - 可以同时选择 `env=production` 和 `env=devel` 的 Pod1
    - 就像说："我想要 'env' 标签是 'production' 或 'devel' 的所有 Pod"
2. **基于标签存在与否进行匹配**：
    - 可以选择所有具有某个标签键的 Pod，不管其值是什么（相当于 `env=*`）1
    - 就像说："只要 Pod 有 'env' 这个标签，不管它的值是什么，我都要"
3. **基于标签不存在进行匹配**：
    - 可以选择所有不包含某个特定标签的 Pod
    - 就像说："我要所有没有 'env' 标签的 Pod"

简单来说，ReplicationController 的选择器很简单，只能做精确匹配，而 ReplicaSet 的选择器更强大，可以进行更复杂的匹配操作，比如"或"关系、"存在"判断等，这使它能更灵活地管理不同类型的 Pod。

ReplicaSet 是 ReplicationController 的下一代产品，功能更强大，最终会完全取代 ReplicationController

  

### 6.3 定义 ReplicaSet

- ReplicaSet 不是 v1 API 的一部分，而是属于 apps API 组和 v1beta2 版本
- 与 ReplicationController 相比，主要区别在于选择器
- 代码示例：

```YAML
apiVersion: apps/v1beta2
kind: ReplicaSet
metadata:
  name: kubia
spec:
  replicas: 3
  selector:
    matchLabels:
      app: kubia
  template:
    metadata:
      labels:
        app: kubia
    spec:
      containers:
      - name: kubia
        image: luksa/kubia
```

![[截屏2025-04-08_20.43.59.png]]

## 7. 总结

Kubernetes 中的 Pod 组织和复制是云计算中容器管理的核心概念。通过将应用程序划分为适当的 Pod 单位，可以实现更好的资源利用和独立扩展。Pod 描述符定义了 Pod 的元数据、规格和状态信息。标签为 Pod 提供了强大的组织功能，允许通过标签选择器进行选择和过滤。为了保持 Pod 的健康，Kubernetes 提供了自动重启机制。ReplicationController 和其更强大的继任者 ReplicaSet 确保始终运行期望数量的 Pod，自动创建替代 Pod 以应对故障情况，并支持 Pod 的水平扩展。这些机制共同构成了 Kubernetes 强大的容器编排能力，使云计算应用程序更加可靠和可扩展。