Question 1(1 point) ~ Saved

How do GFS clients access data stored in the system?

- ﻿By synchronizing data from peer clients using distributed caching
- ﻿﻿By downloading entire files to local storage before processing
- ﻿By retrieving data directly from the master node
- ﻿By requesting file content via a web-based interface
- ﻿﻿By sending all read requests to a metadata service
- ﻿﻿<font color="#ff0000">By communicating directly with chunk servers after obtaining metadata from the master node</font>

Question 2 (1 point) Saved

What was the primary motivation behind the design of the Google File System (GFS)?

- ﻿﻿To provide an encrypted file storage solution
- ﻿﻿<font color="#ff0000">To support large-scale, data-intensive processing workloads</font>
- ﻿﻿To support mobile device file synchronization
- ﻿﻿To replace traditional relational databases
- ﻿﻿To optimize file storage for small, interactive applications
- ﻿﻿To optimize real-time transactional processing

Question 3(1 point) ~ Saved

How does GFS handle file deletions?

- ﻿﻿Clients must explicitly delete data from each chunk server
- ﻿﻿Files are immediately erased from all chunk servers
- ﻿﻿<font color="#ff0000">The master removes metadata, but chunk servers delete the actual data lazily over time</font>
- ﻿The system performs an immediate hard delete for security reasons
- ﻿﻿Files are moved to a "trash" directory before deletion
- ﻿﻿Files are archived in a backup system before deletion

Question 4(1 point) Saved

Which of the following best explains the role of client libraries in the GFS architecture?

- ﻿﻿<font color="#ff0000">They provide an interface for transparent access to GFS functionalities such as read, write, and append</font>
- ﻿﻿They encrypt and decrypt data before storage
- ﻿They directly manage the metadata of files
- ﻿They dynamically adjust the chunk sizes based on file content
- ﻿﻿They are used to control the replication process of chunks
- ﻿﻿They monitor and report the performance of chunk servers

Question 5 (1 point) Saved

What design feature in GFS ensures that concurrent writes to the same file result in consistent file contents?

- ﻿﻿Automatic file merging algorithms
- <font color="#ff0000">﻿﻿Atomic record append operations</font>
- ﻿﻿Distributed locks on every byte
- Client-side conflict resolution
- Serialized client write queues
- Strict version control on file chunks

Question 6(1 point) ~ Saved

What is the primary responsibility of the master node in GFS?

- ﻿﻿Directly handling client file read requests
- ﻿﻿<font color="#ff0000">Managing the file system's metadata and coordinating system-wide operations</font>
- Storing all user data permanently
- ﻿Balancing network traffic between all nodes
- ﻿﻿Encrypting data before storage
- ﻿﻿Managing the hardware resources of chunk servers

Question 7(1 point) Saved

How does GFS typically handle write operations from clients?

- ﻿﻿Clients send data to the master node for direct writing
- ﻿﻿Writes are processed sequentially by a centralized scheduler
- ﻿﻿Writes are immediately committed to disk without replication
- ﻿﻿<font color="#ff0000">Data is written directly to multiple chunk servers in parallel</font>
- ﻿﻿Writes are buffered and later processed by a dedicated write server
- ﻿﻿Write operations are cached on the client side only

Question 8 (1 point) v Saved

How does GFS achieve scalability in its architecture?

- ﻿﻿By partitioning files based on user groups
- ﻿By using cloud-based load balancing only
- ﻿﻿<font color="#ff0000">By having a single master node for metadata and many chunk servers for data</font>
- ﻿﻿By caching all data in a central memory pool
- ﻿﻿By using a peer-to-peer network model with no centralized control
- ﻿By distributing the namespace across multiple master nodes

Question 9 (1 point) Saved

Which component in GFS is primarily responsible for managing metadata, such as namespace and file-to-chunk mapping?

- ﻿The replication manager
- ﻿﻿The metadata proxy
- ﻿The chunk server
- ﻿﻿The client interface
- ﻿﻿<font color="#ff0000">The single master node</font>
- ﻿﻿The distributed lock service

Question 10 (1 point) v Saved

Which of the following is NOT a key design goal of the Google File System?

- Fault tolerance through data replication
- ﻿﻿Scalability across thousands of machines
- ﻿﻿<font color="#ff0000">Efficient support for small, random reads and writes</font>
- ﻿﻿Automatic recovery from component failures
- ﻿﻿High throughput for large data processing
- ﻿﻿Optimizing sequential data access

Question 11 (1 point) Saved

Which statement best describes the role of chunk servers in GFS?

- ﻿﻿They provide encryption services for data in transit
- ﻿﻿They maintain the file namespace and metadata
- ﻿They directly communicate with the master for all operations
- <font color="#ff0000">They store the actual data chunks and handle read/write requests</font>
- ﻿They manage client authentication and access control
- ﻿They balance the overall network load

Question 12 (1 point) / Saved

Why does GFS use relaxed consistency guarantees?

- ﻿﻿To allow better encryption of data
- ﻿﻿To reduce the number of chunk servers needed
- ﻿﻿To optimize performance for real-time applications
- ﻿﻿To ensure that all clients always see the latest version of a file
- <font color="#ff0000">﻿﻿To improve performance and scalability</font>
- ﻿﻿To prevent unauthorized access to files

Question 13(1 point) Saved

In GFS, how are files primarily divided to facilitate storage and management?

- Into variable-sized clusters
- <font color="#ff0000">﻿﻿Into 64 MB chunks</font>
- ﻿﻿Into continuous streams with no division By splitting based on user access patterns
- ﻿﻿Into segments based on file type
- ﻿﻿Into blocks of 4 KB each

Question 14 (1 point) Saved

Why does GFS use a single master node?

- ﻿﻿To eliminate the need for distributed consensus
- ﻿﻿To store both metadata and actual file contents
- ﻿﻿<font color="#ff0000">To centralize metadata management and simplify coordination</font>
- ﻿﻿To balance network traffic between different data centers
- ﻿To allow clients to access data directly without authentication
- ﻿﻿To increase data replication performance

Question 15 (1 point) Saved

How does GFS detect and recover from component failures?

- ﻿﻿Through client-side error correction codes
- ﻿﻿By manual intervention from system administrators
- <font color="#ff0000"> ﻿﻿Through periodic heartbeats and automated re-replication of data</font>
- ﻿By using external backup systems exclusively
- Through a centralized monitoring dashboard that alerts users
- •By storing data only on high-availability hardware