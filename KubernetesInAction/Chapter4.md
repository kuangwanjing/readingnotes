## Chapter 4 Summary

Create ReplicationController to automatically manage the pods.

### Keep pods healthy

1. If the main process crashes, Kubelet knows it and is able to restart the process once it crashes down.
2. If the process doesn't work but doesn't crash down, no restart is triggered so no heal on it. 
3. Even though some errors are caught by applications, some are not(e.g deadlock, infinite loop) catchable, which means health monitoring can not be relied internally. *external monitor*

liveness probes: (not all the application are web application, so Kubernetes provides different ways for liveness probes)

- HTTP probe — response status code
- TCP probe — connection establishment
- Exec probe — execution exit code

Liveness probes are initialized after the pod is created. Remember to set the initialDelaySeconds option to prevent premature probes to avoid unnecessary restart. 

Here is an example from the logs of running of kubia-liveness: 

> Events:
>   Type     Reason     Age                   From               Message

----     ------     ----                  ----               -------
>  Normal   Scheduled  27m                   default-scheduler  Successfully assigned default/kubia-liveness to minikube
>   Normal   Created    23m (x3 over 26m)     kubelet, minikube  Created container kubia
>   Normal   Started    23m (x3 over 26m)     kubelet, minikube  Started container kubia
>   Normal   Killing    21m (x3 over 25m)     kubelet, minikube  Container kubia failed liveness probe, will be restarted
>   Normal   Pulling    21m (x4 over 27m)     kubelet, minikube  Pulling image "luksa/kubia-unhealthy"
>   Normal   Pulled     21m (x4 over 26m)     kubelet, minikube  Successfully pulled image "luksa/kubia-unhealthy"
>   Warning  BackOff    7m22s (x26 over 14m)  kubelet, minikube  Back-off restarting failed container
>   Warning  Unhealthy  113s (x28 over 25m)   kubelet, minikube  Liveness probe failed: HTTP probe failed with statuscode: 500

And list the pods to see the times of restart of kubia-liveness count up to 11 because since the pod is created, the liveness probes are initialized. And after 5 probes, the process signals a KILL signal. Restart is triggered. 

> kubectl get pods
> NAME             READY   STATUS    RESTARTS   AGE
> kubia-liveness   1/1     Running   11         33m

**Principles of creating liveness probes:**

- Specify a particular request for liveness probes.
- Do not let external factors to influence the probes, only to check the app internally. Go example is the probes of frontend web server shouldn't be influnced by the availability of backend databases. 
- Probes shouldn't use too much computational resources because probing is counted as the CPU time of the container, which may be limited among the cluster.
- Don't  bother to retry in probes.

### ReplicationController

controller's reconciliation loop: depends on label selector, replica count, pod template

A ReplicationController’s job is to make sure that an exact number of pods always matches its label selector:

1. Failed node replacement 
2. Missing pod replacement
3. Scaling horizontally or manually

#### Maintain the state

In the exercise of moving the pod out of scope by changing its label, the ReplicationController detects that the number of pods with the label it cares about does not match with the configuration, then it spins up another new pod with the pod template. The replicationController always maintains the correct state. 

<u>Note:</u> This ensures consistency of the Kubernetes cluster to some extent because user of Kubernetes never tells Kubernetes cluster to do something but tell it what is the right state. Kubernetes reacts to maintain the state whenever the environment changes.  

*Question:* Does this mean Kubernetes is only good for applications that are stateless? How to ensure that scaling and de-scaling does not affect the availability of the system? How to scale the application like database?  

### ReplicaSets

New replacement of ReplcationController, more flexible on dealing with labels with more expressive label selectors. 

### DaemonSets

Run one pod on each node. 

Application: log collector, monitor … 

### Running completable task

ReplicationController, ReplicaSets and DaemonSet are all considered to run continuous task. Once the task stops, it is restarted by the Kubernetes cluster. 