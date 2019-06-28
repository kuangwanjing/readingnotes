# Dynamo 

Introduction:

* highly available key-value data storage system ("always writable")
* highly scalable
* manage the trade-off between consistency and availability 

Assumption:

* Data model: 
  * aggregated oriented
  * Key-value model
  * store relatively small object (< 1MB)
* Efficiency:
  * SLA(service level agreement): value latency of  the $99.9^{th}$ percentile requests instead of the average
* The data storage system runs on non-hostile environment.

Design Consideration:

* Replication: Although the synchronous data replication provides a strong consistent data access, but this doesn't guarantee the high availability and harm the user experience. So Dynamo is designed to have eventual consistency and apply asynchronous replication.
  * When to perform the process of resolving update conflicts? Traditionally during writes but this harms the write availability. So it is during reads that conflicts are resolved. This requires some conflict resolving algorithm.
  * Who to perform the process? Application or data storage? Not many choices data storage has — "last-write wins" but application can have flexible strategies to merge the conflicts. 
* Scalability
* Symmetry and Decentralization: all the nodes should have the same responsibilities which helps to maintain the whole system(hundreds of nodes so this feature is great to release the maintainance burden).
* Heterogeneity: different machine can be added as a node regardless of their disk capacity or CPU resource. This is achieved by virtual node design (virtualization) .

System Architecture & Implementation:

* Interface: get() and put()
  * data structure: key - context (meta-data) - value (the meta-data contains version and other updating information that helps to resolve the conflict), key is hashed into a 128-bit identifier.  
* Partitioning Algorithm
  * Consistent hashing: 
    * circular array, changing one node only affect its neighbors but not other nodes. 
    * but the distribution of keys may not be even and consider the heterogeneity, virtual nodes are used. A physical nodes is mapped to multiple virtual nodes.
  * Strategies envolved:
* Replication
  * Determine N which is the number of copy of a object. (N is usually set to be 3)
  * A lists of nodes (N nodes) store an object is called the preference list for that key. 
  * The key falls into the range calculated by the hash function and choose other nodes by skipping positions of the ring.
* Data Versioning
  * Why versioning: the replication happens asynchronously, thus update conflicts are not uncommon. The versioning tell whether conflicts exist for an object among the replication.
  * How: vector clocks(not timestamp because timestamp can't tell those ->), parallel branches or having causal order.  the size of vector can grow too long, occupping to much storage space => limit the size but this case is not common. 
* Execution of get() and put():
  * coordinator: A node handling a read or write request.
    * Deciding which node to handle the operation: Here a load balancer may select any one node to handle the request but if that node is in the top N list, it redirect the request to the first node in the list. 
    * How to operate? Event-driven: use one state machine to handle one request and sub-task of handling a request is operated in each state
      * for read, the state machine wait for a while after all is done to receive any stale version of the nodes and perform read-repair to those nodes.
  * Quorum System: N, W, R so that W + R > N, where W and R are set to be less than N to get lower latency.
  * SLA consideration: sometimes popular keys are frequently read or written so that the load of the system is not evenly distributed.  the coordinator for a write is chosen to be the node that replied fastest to the previous read operation(in the context) — "read-your-write" consistency
* Handling Failures  — Hinted Handoff
  * Sloppy quorum: The write operation is usually handled on the first N nodes. But if those are not available temporarily, the operaton is handled by the next node and the update is synchronized back to those nodes when they recover. 
  * handling transient failures 
  * This event is written into the metadata of the object. (Context)
* Handling permanent failures — Replication synchronization
  * anti-entropy
  * Goal: to detect the inconsistencies between replicas faster and minimize the data transferred.
  * Merkle tree 
    * hash tree
    * each node is comparable by its hash value
    * detecting the inconsistencies by comparison the tree recursively. 
  * the node maintains a tree for each key range
  * but updating a key requiring the tree to be recalculated. 
* Membership and Failure Detection
  * Membership Discovery
    * the administrator can add or remove a node.
    * the change of membership is propagated through gossip-based protocol and the view of membership of each node eventually become consistent.
    * the nodes persist the change.
  * External Discovery
    * to prevent logical partition 
    * seed nodes play a role of reconciling the partition
    * seed nodes are known to all of the nodes and they are fully functional nodes in the ring.
  * Failure Detection
    * Gossip 
    * Not sacrificing the client's requests to do failure detection
* Storage Nodes
  * Adding/Removing 
    * a set of tokens(ranges in the ring) is assigned to the node
    * bootstrapping of the new node: keys are transferred from the nodes which no longer are responsible to those ranges to the new node.
    * Removal: reverse allocation of keys happens. 
  * Data storage
    * Different storage engines are pluggable to the node to meet different requirements.

Experience

* Reconciliation choice:
  * business logic specific reconciliation 
  * Timestamp reconciliation — "last-write-wins"
* Balancing Performance and Durability
  * N, W, R are configuarable. 
* Ensuring Uniform Load distribution
  * "Out-of-balance" nodes — there are more such nodes in low loads. (During high loads, the popular keys are evenly distributed among the nodes)
  * Partition Strategies:
    * T random tokens per node and partition by token value: new node has to "steal" the keys from the original nodes and the transferring of keys are in low-priority. This makes the bootstrapping time-consuming, worse during high loads. Recalculating the Merkle tree is non-trivial operation. No way to snapshot the system due to the randomness.
    * T random tokens per node and equal sized partitions: Q equally sized partitions over the universal key space. Q >> N and Q >> S * T. decoupling of partitioning and partition placement, and enabling the possibility of changing the placement scheme at runtime.
    * Q/S tokens per node, equal-sized partitions: each node takes Q/S partitions. each partitions can be stored as a separated file -> faster bootstrapping ; easy archival;