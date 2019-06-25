#Managing pods’ computational resources

### Requesting resources for a pod's containers

The requests are specified for each container of a pod(specified in the pod's specification).

Resources include: 

* CPU: unit - millicore, 1 core = 1000 millicore
* Memory
* Other customized resource (GPU … )

Requesting resources guarantees that containers of pod get enough resources to run but it doesn't limit the resources they use.

####Resource requests affect the pod scheduling

Before scheduling a pod, Kubernetes filters the nodes with enough resources requested by the pod. Whether a node has enough resources is determined by the total resources the pods request which are running on it rather than the actual used resources.

Given some nodes, which one is the best to be scheduled? 

* LeastRequestedPriority: prefers nodes with fewer requested resources (with a greater amount of unallocated resources)
* MostRequestedPriority: prefers nodes that have the most requested resources (opposite)

The MostRequestedPriority policy is often used to make the deployment tight enough to save the cost on the hardware.

#### Resource requests affect how CPU time is shared

Proportionate to the ratio of requested CPU of the pods. 

### Limiting resources available to a container

CPU is compressible resource while memory is incompressible resource.

Resource limits are not constrained by the available resource amounts -> it does not affect the pod scheduling. But if memory of a node is used up, certain pods should be killed to free some resources. (OOMKill) Then the pod enters CrashLoopBackoff status (try to restart with exponential backoff algorithm)

#### Understanding how apps in containers see limits

App is not aware of the container's limits. So for JVM, it asks for more memory from the OS than it is limited to use since the available physical memory is larger than the the one from the resource limit.

The same thing happens to the CPU resource limits. It may make the app creating too many threads to competing the limited CPU time.!!! Solution: Get the limits from downward API and let the app to get the CPU resource from the configuration.

### QoS Classes

These classes are used to determine which pod is to be terminated when resources are not enough.

* BestEffort
* Burstable
* Guaranteed

The pods are compared with the OOM scores when choosing the pods from the same class.

score = memory it used / memory it is limited. The higher it is, the sooner it is terminated. 

### Default requests and limits can be set

LimitRange resource

###Limiting the total resources available in a namespace

ResourceQuota resource: disk (HDD, SSD)

Classes (scope the quota applies to):

* BestEffort
* NonBestEffort
* Terminating
* NonTerminating

​	

