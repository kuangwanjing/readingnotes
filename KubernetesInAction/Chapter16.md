## Advanced Scheduling

* taints and tolerations
* Node affinity rules
* pod affinity
* pod anti-affinity



### Using taints and tolerations to repel pods from certain nodes

select which nodes a pod can or can’t be scheduled to by specifically adding that information to the pod.

allow rejecting deployment of pods to certain nodes by only adding taints to the node without having to modify existing pods.

* Taint(something bad about the nodes) : key/value:effect(NoSchedule/PreferNoSchedule/NoExecute)
  * Custom taint: this can be used to make segregation of production and non-production environment where developing version will never be deployed to the production nodes.
* Tolerations(indicate that a pod can tolerate the tainted nodes):
  * Key/Operation/Value/Effect

Nodes can have more than one taint and pods can have more than one toleration. 

Application:

* define unpreferred nodes or even evict pods from nodes
* configuring how long after a node failure a pod is rescheduled.

### Using node affinity to attract pods to certain nodes

Question: Why not define node selector in the pod specification for scheduling?

Answer: Node selector is not expressive enough. The node had to include all the labels specified in that field to be eligible to become the target for the pod.

Node affinity rules are more expressive since it is not *a hard requirement*.

affinity currently only affects pod scheduling and never causes a pod to be evicted from a node. 

Another admission plugin!

Node affinity help prioritize nodes when scheduling pods. Multiple weighted rules can be combined to affect the scheduling. 

### Co-locating pods with pod affinity and anti-affinity

When the Scheduler is deciding where to deploy a pod, it checks the pod’s pod-Affinity config, finds the pods that match the label selector, and looks up the nodes they’re running on.

Example: a frontend pod prefers to co-locate with the backend pod to reduce the latency. 