## Aurora: Design Considerations for High Throughput Cloud-Native Relational Database

Background:

* The I/O bottleneck faced by the traditional database systems changes under distributed environment. The I/Os are spread across many nodes and many disks in a multi-tenant fleet. The bottleneck has moved to the packets per second and bandwidth in the systems.
* Some synchronized operations are inevitable and due to the miss in the database buffer, these operations are in the risk of resource contention, context switch. 
* The 2-phase commit protocols of commiting transcation are not fault-tolerant(failures). 

 How does Aurora try to solve these problems:

* Principles:
  * Leverage the redo log more aggressively.
  * Use a service-oriented architecture with multi-tenant storage. 
* Architecture:
  * Build storage as an independent fault-tolerant and self-healing service across multiple data centers.
  * Only write redo log records to storage, and this reduces the network IOPS(input-output opertions per second) by an order of magnitude. => remove a major bottleneck.
  * Move expensive operations like backup, and redo recovery from time-expensive operations in **database engine** to continuous asynchronous operations amortized across a large distributed fleet.

Durability at scale:

* Replication and correlated failures:
  * decouple storage tier from compute tier. Disk can fail too but this can be solved by replication.
  * Quorum-based voting protocols are used. V_r and V_w > V. And V_w > V / 2. The common approach is: V_r = V_w = 2/3 V. 
  * 2/3 quorum is believed to be inadequate. If one AZ fails, any failure of the othe two zones breaks the quorum voting. In Aurora, 3 AZs are used with two replicas in each AZ. That is data is copied in 6 ways. 
* Segmented Storage:
  * To ensure the availability, it is either to reduce the MTTF or the MTTR. It is hard to reduce MTTF so it is to reduce the MTTR to shrink the window of vulnerability. 
  * Data is segmented into Protection Groups in size of 10GB. PGs are supported by a large number of storage nodes. 
  * Since the network link is high performant, it takes seconds to transfer a 10GB file from one node to another. 
* Operation advantages of resilience:
  * Heat-management: if one of the segments is on a hot disk or bad one, Operators can mark it as a failure and quorum quickly migrate the data to cooler nodes.
  * OS and security patching: brief unavailability event => take turns to be applied to different nodes.
  * Same thing for software upgrade.

The Log is the Database:

* The burden of amplied writes: 
  * the more replication of the data, the write events are spread across the replicas. Then a single write is amplied as multiple writes. PPS increases. (Log, bin log, data, double-write, FRM files...)
  * Synchronization stalls the pipeline and dilate lantencies. 
* Offloading redo processing to storage:
  * Traditional database generates a redo log and invokes a log applicator to apply the redo log. Transaction commit requires the log to be written but the data page write may be deferred. 
  * In Aurora, redo log records are pushed to storage tier but not database and the data pages are generated in background.  Then the packages for replication among database replicas are alleviated. (Offloading!)
  * Checkpoint, backup are optional in the background. 
  * This allows the system to aggressively replicate data for durability and availability and issue requests in parallel to minimize the impact of jitter.
  * For crash recovery, the page is materialized using durable redo records so the recovery is brought to the foreground processing(which is in high priority so the recovery is fast enough)
* Storage Service Design Points:
  * Tenet: to minimize the latency of foreground write requests, move the majority of storage processing to the background.  
  * Background processing has negative correlation with the foreground processing. (Throttle)

The log marches forward:

* Solution sketch: asynchronous processing
  * Logs advance as a sequence of changes. Each log record associates with a Log Sequence Number(LSN).
  * Since nodes may lose some of the log records, they can gossip with other nodes to fill the gap based on the PGs they maintain. (Just read segements from the nodes rather than quorum read)
  * If the node crashes, the storage does its own recovery before the database restarts, ensuring the databases see the same storage volume. 
  * The storage service determines the highest LSN for which it can guarantee availability of all prior log records(VCL or Volume Complete LSN). (During recovery, records with LSN larger than the VCL must be truncated.)
  * The database verifies the highest Consistency Point LSNs. VDL is the highest CPL that is durable. Logs with high CPLs larger than VDL must be truncated. (Why use VDL not CPL as the basis of truncating? — because VDL is persistent, CPL is the status of database engine but the engine is not attached directly to persistent volume.)
* Normal Operation
  * Writes:
    * Database advances its VCL when getting acknowledgment on write quorum from storage service. 
    * LSN is allocated by the database and some contraint is put on the value of LSN. (A large LSN can throttle the incoming writes)
    * Each segment sees a subset of log records and each has its own Segment Complete LSN(SCL) for that PG.
  * Commits:
    * Transactions are identified by the completing LSN and are handled by a specific thread. As the VDL is advanced, the waiting transactions are committed.
  * Reads:
    * Traditionally, if read on the buffer is missed, the system finds the page from disk and may evict a page out of the buffer and bring that page to the buffer.
    * But Aurora never writes out page to disks because it ensures the page in the buffer is always in the lastest version. It onlys evicts the page with LSN larger than or equal to the current VDL. 
    * The read request does not need to establish consensus with read quorum under normal circumstances. Get the read-point(VDL) when the page is read, find the segment that has sufficient data, that is its Segment Complete LSN is greater than VDL. 
    * "Low water mark" based on the read-points of all PGs. The log records lower than the low mark(per-PG Minimum Read Point LSN), then it is safe to marterialized pages on disk by coalescing the older log records and then garbage collecting them. 
  * Replicas:
    * To minimize lag, the log stream generated by the writer and sent to the storage nodes is also sent to all read replicas. 
    * The reader updates the pages in the buffer if the log records refer to pages in the buffer. 
    * The reader only applies those records whose LSN is less than VDL. 
* Recovery:
  * Checkpointing is expensive and interfer with foreground transactions. 
  * Recovery is taken place synchronously with requests when it calculates the page with durable records. 
  * Before reestablishment of database, it needs to restore the runtime state by calculating its PGs VDL  from segments that meets the write quorum. Higher log records are truncated. 

The complete Architecture:

* The write operations trigger the database engine to generate logs with LSN and update the page in buffer.
* Only when the log records are durable to disk, write requests are done. 
* The write to pages are done in background. 
* Host Manager is used to monitors a cluster's health and determines if it needs to fail over or replace the nodes. 

Lessons I learned: 

* Learn the main bottlenecks of the system. 
* This is a service-oriented architecture which combines the storage, database engine, database service, quorum control, log streaming and some other assistant components.
  * Storage: decouple storage from compute(engine). The use of storage tier is resilient and scalable.(Protection Groups, replication and backup of groups)
  * Database Engine: low level database reads and writes. Manages data in pages and exposes pages to storage service. 
  * Database Service: control the runtime state of database instance. Apply log records to the storage, checkpoint, recover from crashes, manage the buffer. Persist the log records. 
  * Quorum control => 6-way replication. 2/3 quorum for both read and write. 
  * Log streaming: protocols on defining the transactions and state of nodes. 