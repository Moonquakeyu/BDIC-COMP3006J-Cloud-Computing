Question 1 (1 point) v Saved

What is the primary purpose of Borg?

- <font color="#ff0000">﻿﻿To orchestrate and schedule jobs across Google's data centers</font>
- ﻿To manage Google's large-scale distributed storage systems
- ﻿﻿To optimize database queries in cloud environments
- ﻿﻿To balance web traffic across Google services
- ﻿﻿To secure Google's network infrastructure
- ﻿﻿To provide a public cloud service for external customers

Question 2 (1 point) v Saved

What are the key responsibilities of Borg's scheduler?

- ﻿﻿Providing high-speed interconnects for distributed databases
- ﻿﻿Managing physical data center cooling systems
- ﻿﻿Ensuring all jobs are executed in a strict sequential order
- ﻿﻿Managing Google's Al and machine learning models exclusively
- ﻿﻿Ensuring data encryption and enforcing access control
- ﻿﻿<font color="#ff0000">Allocating compute resources, managing job execution, and handling failures</font>

Question 3 (1 point) Saved

What is a Borgeell?

- A network partitioning mechanism
- A specialized scheduling algorithm within Borg
- ﻿﻿<font color="#ff0000">A collection of machines managed by a single Borg master</font>
- ﻿A distributed file system for storing application data
- ﻿﻿A backup cluster used for fault tolerance
- ﻿﻿A lightweight virtual machine used in Borg

Question 4 (1 point) v Saved

How does Borg handle job failures?

- It discards failed jobs and notifies the user
- It transfers jobs to a backup cluster in another region
- <font color="#ff0000"> ﻿﻿It automatically restarts failed jobs based on predefined policies</font>
- ﻿﻿It schedules jobs only on machines with no prior failures
- ﻿﻿It replicates jobs across multiple data centers for redundancy
- ﻿﻿It pauses execution and waits for manual intervention

Question 5 (1 point) v Saved

How does Borg support priority-based scheduling?

- ﻿By dynamically adjusting job priorities based on execution time
- ﻿﻿<font color="#ff0000">By assigning priority levels to jobs and preempting lower-priority tasks if needed</font>
- ﻿By manually reordering jobs in the queue
- ﻿By assigning equal resources to all jobs regardless of priority
- ﻿By enforcing a strict first-come, first-served job execution order
- ﻿﻿By allowing only high-priority jobs to run on dedicated machines

Question 6 (1 point) Saved

What component of Borg is responsible for tracking job and task states?

- ﻿The Job Monitor
- ﻿The Borg Database
- ﻿﻿The Priority Queue Manager
- ﻿﻿The Borg UI
- <font color="#ff0000">﻿﻿The Borgmaster</font>
- ﻿﻿The Node Agent

Question 7 (1 point) Saved

What is the main advantage of Borg's job preemption mechanism?

- ﻿﻿It prevents jobs from being scheduled on underutilized machines
- ﻿﻿It prevents any resource contention among tasks
- ﻿﻿It ensures no task ever gets interrupted
- <font color="#ff0000"> ﻿﻿It ensures high-priority jobs get resources even if the cluster is full</font>
- ﻿﻿It guarantees all jobs complete within a fixed time
- ﻿﻿It allows users to manually prioritize their tasks at runtime

Question 8 (1 point) Saved

How does Brog handle resource overcommitment?

- ﻿By strictly enforcing fixed resource reservations for each job
- <font color="#ff0000">﻿﻿By allowing multiple tasks to share resources and dynamically adjusting allocations</font>
- ﻿By scheduling jobs only when enough spare capacity is available
- ﻿﻿By enforcing a hard limit on memory and CPU usage per task
- ﻿By reserving extra nodes for emergency resource allocation
- ﻿By limiting the number of jobs per user

Question 9 (1 point) Saved

What are the two main fypes of workloads in Borg?

- ﻿﻿Video processing and cloud storage services
- ﻿﻿<font color="#ff0000">Long-running services and batch jobs</font>
- ﻿Real-time streaming and transactional workloads
- ﻿﻿Machine learning models and data processing tasks
- ﻿Web applications and distributed databases
- ﻿﻿Interactive queries and background processes

Question 10 (1 point) Saved

How does Borg enable efficient cluster utilization?

- ﻿﻿<font color="#ff0000">By allowing fine-grained resource sharing and overcommitment</font>
- ﻿By enforcing strict CPU and memory isolation without sharing
- ﻿By restricting certain workloads to specific nodes
- ﻿By reserving large portions of cluster capacity for specific applications
- ﻿By dynamically shutting down idle machines
- By limiting the number of jobs running concurrently

Question 11 (1 point) Saved

What component in Borg is responsible for making scheduling decisions?

- The Load Balancer
- The Distributed Task Handler
- The Metadata Store
- The Node Agent
- <font color="#ff0000">﻿﻿The Borgmaster</font>
- ﻿﻿The Replica Manager

Question 12 (1 point) Saved

How does Borg schedule tasks onto machines?

- ﻿By allowing users to assign tasks to machines manually
- <font color="#ff0000">﻿﻿By using a two-phase scheduling mechanism</font>
- ﻿By using an Al-based learning scheduler
- By first placing all high-priority tasks and then filling the remaining slots with low-priority tasks
- ﻿﻿By following a strict round-robin placement algorithm
- ﻿By pre-allocating resources based on static configurations

Question 13 (1 point) Saved

What technique does Borg use to maintain high availability of its master components?

- Relying on an external cloud-based service to manage failover
- ﻿﻿Running multiple redundant Borgmasters in an active-active mode
- ﻿﻿Keeping a static failover master that requires manual activation
- Distributing Borgmaster functionality across all worker nodes
- ﻿﻿<font color="#ff0000">Using a primary-backup approach with Paxos-based leader election</font>
- Restarting the master components from snapshots when a failure occurs

Ouestion 14 (1 point) Saved

Why does Borg support task packing on machines with overcommitted resources?

- To prevent machines from becoming idle
- ﻿﻿To distribute workloads equally across all data centers
- ﻿﻿To allow users to manually adjust resource allocations
- ﻿﻿To guarantee that tasks always receive their requested resources
- ﻿To ensure that high-priority tasks always get the best performance
- ﻿﻿<font color="#ff0000">To increase cluster utilization and efficiency</font>

Question 15 (1 point) Saved

How does Borg handle dependencies between tasks within a job?

- By using a built-in dependency resolution framework called BorgSync

- By enforcing a strict sequential execution of tasks in the job queue

- <font color="#ff0000">By allowing tasks to express dependencies using a DAG (Directed Acyclic Graph)</font>

- By pre-allocating all resources before executing the first task

- By relying on users to manually schedule dependent tasks 
- By running all tasks in parallel without dependency