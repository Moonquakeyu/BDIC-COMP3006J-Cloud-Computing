## 问题1: In GFS, how are files primarily divided to facilitate storage and management?

**正确答案: Into 64 MB chunks**

GFS将文件分割成固定大小的64MB块(chunks)。这是GFS的基本存储单位，每个chunk在多个chunkserver上有多个副本。选择这个大小是为了减少与master的交互频率和减少master上存储的元数据量。

## 问题2: Why does GFS use a single master node?

**正确答案: To centralize metadata management and simplify coordination**

GFS使用单一master节点来集中管理元数据并简化协调工作。master维护所有文件系统元数据，包括命名空间、访问控制信息、文件到chunk的映射以及chunk的当前位置。这种集中式设计简化了复杂的数据放置和复制策略的实现。

## 问题3: Which of the following is NOT a key design goal of the Google File System?

**正确答案: Efficient support for small, random reads and writes**

GFS并不是为高效支持小型随机读写而设计的。它主要针对大文件的顺序访问和追加写入进行了优化。GFS的设计假设包括文件通常很大，主要有两种读取模式（大批量顺序读取和小型随机读取），写入通常是追加而非覆盖。小型随机读写效率不是GFS的主要设计目标。

## 问题4: Why does GFS use relaxed consistency guarantees?

**正确答案: To improve performance and scalability**

GFS使用宽松的一致性模型以提高性能和可扩展性。这种设计简化了系统实现，减少了维护严格一致性所需的协调开销，同时满足了大多数应用的需求。GFS应用可以通过使用追加而非覆盖、使用检查点和写入自验证记录等方式来适应这种一致性模型。

## 问题5: How does GFS handle file deletions?

**正确答案: The master removes metadata, but chunk servers delete the actual data lazily over time**

GFS采用延迟删除机制。当文件被删除时，master首先将其重命名为隐藏名称，然后在定期垃圾回收过程中才真正删除这些文件的元数据和相关chunks。这种垃圾回收机制简化了系统设计并提高了系统可靠性，允许系统在后台逐步回收存储空间。

## 问题6: What was the primary motivation behind the design of the Google File System (GFS)?

**正确答案: To support large-scale, data-intensive processing workloads**

GFS的主要设计动机是支持大规模、数据密集型的处理工作负载。它是为满足Google日益增长的数据处理需求而设计的，特别是需要处理大量大文件的应用场景，如网页索引、日志处理和数据分析等。GFS的设计目标包括高性能、可扩展性、可靠性和可用性，特别针对大规模数据处理进行了优化。

## 问题7: How does GFS achieve scalability in its architecture?

**正确答案: By having a single master node for metadata and many chunk servers for data**

GFS通过单一master节点管理元数据和多个chunkserver存储实际数据的架构实现可扩展性。这种架构分离了控制流和数据流，master只处理轻量级的元数据操作，而数据直接在客户端和chunkserver之间传输，避免了master成为系统瓶颈。

## 问题8: Which statement best describes the role of chunk servers in GFS?

**正确答案: They store the actual data chunks and handle read/write requests**

Chunkserver在GFS中的主要角色是存储实际的数据块(chunks)并处理读写请求。它们将chunk存储为本地Linux文件，并通过chunk句柄(handle)和字节范围指定的方式处理对chunk的读写操作。每个chunk通常在多个chunkserver上有多个副本以保证可靠性。

## 问题9: How does GFS typically handle write operations from clients?

**正确答案: Data is written directly to multiple chunk servers in parallel**

在GFS中，客户端获取元数据后会直接将数据写入到多个chunkserver中。写入过程通常包括：客户端首先向所有持有相关chunk副本的chunkserver推送数据；然后客户端向primary chunkserver发送写请求；primary确定写入顺序并将操作应用到本地，同时将操作转发给其他副本；所有副本完成操作后向primary回复；primary向客户端回复操作结果。

## 问题10: How do GFS clients access data stored in the system?

**正确答案: By communicating directly with chunk servers after obtaining metadata from the master node**

GFS客户端首先从master获取文件的元数据信息，包括文件到chunk的映射和chunk的位置，然后直接与存储实际数据的chunkserver通信以读取或写入数据。这种设计将控制流与数据流分离，减轻了master的负担。

## 问题11: How does GFS detect and recover from component failures?

**正确答案: Through periodic heartbeats and automated re-replication of data**

GFS通过定期的心跳消息检测组件故障，并通过自动重新复制数据来恢复。当chunkserver故障时，master会注意到心跳消息的缺失；如果某个chunk的副本数量低于设定的复制因子，master会指示其他chunkserver创建新的副本。此外，GFS还使用校验和检测数据损坏，并从有效副本中恢复数据。

## 问题12: What is the primary responsibility of the master node in GFS?

**正确答案: Managing the file system's metadata and coordinating system-wide operations**

GFS中master节点的主要职责是管理文件系统的元数据并协调系统范围的操作。元数据包括文件命名空间、访问控制信息、文件到chunk的映射以及chunk的当前位置。系统范围的操作包括chunk租约管理、垃圾回收和chunk迁移等。

## 问题13: Which component in GFS is primarily responsible for managing metadata, such as namespace and file-to-chunk mapping?

**正确答案: The single master node**

在GFS中，单一的master节点负责管理元数据，如命名空间和文件到chunk的映射。这种集中式的元数据管理简化了系统设计并提高了协调效率。master保持所有元数据在内存中，并通过操作日志和周期性检查点进行持久化。

## 问题14: Which of the following best explains the role of client libraries in the GFS architecture?

**正确答案: They provide an interface for transparent access to GFS functionalities such as read, write, and append**

GFS客户端库提供了一个接口，使应用程序能够透明地访问GFS功能，如读取、写入和追加。它们实现了与master和chunkserver通信的协议，处理元数据缓存，以及管理数据流。客户端库隐藏了分布式文件系统的复杂性，向应用程序提供类似文件系统的API。

## 问题15: What design feature in GFS ensures that concurrent writes to the same file result in consistent file contents?

**正确答案: Atomic record append operations**

GFS的原子记录追加(atomic record append)操作确保了对同一文件的并发写入产生一致的文件内容。在这种操作模式下，客户端只指定数据而不指定偏移量，GFS在文件末尾追加数据并保证原子性。这种设计特别适合多生产者/单消费者模式，允许多个客户端同时向同一文件追加数据而不需要额外的同步机制。