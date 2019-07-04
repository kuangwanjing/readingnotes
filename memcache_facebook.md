## Scaling Memcache at Facebook

### Contributions:

* basic Memcache architecture.
* Improve the performance of memcache and increase memory efficiency.
* Scale the whole system
* Characterize the workloads and the method to analyze the workloads

### Overview

* Preconditions:
  * User consume an order of magnitude more content than they create. ( *reads >> writes* )
  * The read operations fetch data from a variety of sources such as MySQL, HDFS. (*The backend is heterogeneity, it is necessary to protect the backend by Memcache.*)
  * Key-value data model
  * Basic operations: get(), set() and delete()

* Query cache:
  * Cache Pattern: demand-filled look-aside cache(cache aside)
  * Delete the key in the cache to invalidate the cache when data is updated at the backend storage. Why not use set to update the cache? Race condition exists for this cases if using set():

  ​       Cache                 	T1  				T2       			

  ​		Miss <———————————read key

  ​															read old data

  ​								 updata db

  ​		new data<—— set key	

  ​	 old data <——————————update cache  

  Delete is idempotent. Using delete can attach a key with some state that can tell reads that the key should be evicted and something special should be done.  

* Generic cache:
  * general key-value store
  * no server-to-server coordination(avoid incast congestion)
  * Fan-out: a page involves a lot of reads towards even a hundred of keys. 
  * Address data replication between clusters.

### In a Cluster: Latency and Load

* Reducing latency:
  * Scalable in a cluster: Keys are distributed across the servers through consistent hashing. Problems: this all-to-all communication can cause incast congestion or allow a single server become the bottleneck.(Data replication)
  * Memcache client: build memcache client dealing with serialization, compression, request routing, error handling(e.g transient failure) and request batching. Since the memcache client directly deals with the cache operations, so they have enough information to optimize the operations and handle errors. (Good design!)
  * Parallel requests and batching: requests can be considered as a DAG(read a key, get the subkeys, and fetch the subkeys, so read dependency lies in the page request). Parallel reads. The same reads can be batched. These all alleviate the network overhead in the cluster.
  * Clients are stateless, limit the use case so performance can be optimized.
  * Rely UDP for gets and TCP for sets. (<u>high read -> relatively unreliable but lighweight protocol, low write -> reliable but heavyweight protocol</u>). The speed of sending out TCP packets is controlled by TCP's congestion control. What about UDP packets? — similar sliding window mechanism is used here.
* Reducing load:
  * Leases: 64-bit token bound to the specific key
    * Stale sets and thundering herds harm the performance and increase the load of backend.
    * With lease, the memcached can verify and determine whether the data should be stored and arbitrate concurrent writes. 
    * A modified lease regulates the rate at which it return tokens. A read request can be waited by special signal from memcached. 
  * Memcache pools: catergorize the memcache keys into several patterns and manage them in different pools, scale differently for different pools. Load balancing. 
    * default(wildcard)
    * app customized
    * high frequently accessed
    * Low frequently accessed
  * Handling Failures:
    * Failures:
      * a small numbers of hosts are inaccessible 
      * a widespread outage that affects a significant percentage of the servers. 
    * For small outages: Gutter hosts substitute the inaccessible hosts and reduce the potential increased loads toward backend, avoiding cascading failures. (The Gutter kinda works like a buffer)
    * For widespread outage: Replication.

### In a Region: Replication

* Regional Invalidations: Mcsqueal guards the invalidations.
  * Reduce packets rate between clusters: The commit logs of database are propagated across clusters and Mcsqueal intercepts the logs synchronization packets and delete the corresponding keys (then the keys of DB couples with the keys of cache?), it also allows the replay of the logs. 
  * Invalidation via web servers(memcache clients): allows routing, handling errors. 
* Regional Pools: Reliability, over-replication of infrequently accessed keys is not memory efficient. 
  *  now heuristic 

* Cold Cluster Warmup
  * During warm-up, the hit rate is poor — badly affect the performance of caching. 
  * A request to a cold cluster is directed to a relatively hot cluster.(second hold-off)

### Across Regions: Consistency

* Regions across geography
* Master-Slave model: one master multiple slaves.
* Consistency between master and slaves: 
  *  DB updates of slaves may lags behind the master.
  * 

 



​															