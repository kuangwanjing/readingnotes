## Automatic scaling of pods and cluster nodes

This chapter focuses more on the horizontal scaling 

Why manual scaling is not enough? — Because it requires the administrators to know the load in advanced or the load changes gradually over time. It can not help with unpredictable changes.

### Horizontal pod autoscaling

HPA(HorizontalPodAutoscaler) — obtains metrics of all the pods managed by this object, calculated the required numbers of pod by the metrics and its requirement, then update the replicas of the pods. 

Metrics are collected by cAdvisors in each pod and gathered by the Heapster. The HPA obtains the metrics by API. 

How to calculate the required number of pods?

​	max (sum of metric / required metric ) for each kind of metric (CPU, QPS)

Autoscaler exposes it functionality towards these objects:

* Deployments
* ReplicaSets
* ReplicationControllers
* StatefulSets

Metrics are collected periodically. The end effect is that it takes quite a while for the metrics data to be propagated and a rescaling action to be performed. 

Autoscaling can not scale the application up to infinite numbers of pods or down to 0 pod. 

Memory is not a good metrics for scalling because scaling the pods doesn't guarantee the drop of memory use. The application must release the memory by itself.

Three kinds of metrics that can be defined in the HPA:

* Resource — the resource the pod is using
* Pods — any non-resource metric related to the pod
* Object — cluster-level metric, for example the lantency of a service(ingress)

### Vertical pod autoscaling

Autoscale the cluster