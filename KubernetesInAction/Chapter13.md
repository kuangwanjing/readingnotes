## Securing cluster nodes and the network

### Using the node's namespace in a pod

Network interface is a namespace-level resource of the node, which only exports the default namespace network interface. The pod can use the default's network interface rather than the virtual network namespace(other namespaces) by setting:

> hostNetwork: true

1. Host

   Usage: When Kubernetes Control Plane is deployed as pods which needs to manipulate the nodes' resource, it becomes effective to make an illusion that the pod is actually using the actual resource. 

2. Port number

   Only one such pod is scheduled on one node since no two process can share the same port number.

   DaemonSet

3. PID, IPC namespace

   share the same pid space and inter-process communication of the default namespace of the node.

All of the above sharing can be achieve by setting the spec of a pod.

### Configuring the container’s security context

Setting the securityContext property

1. Defining what userID a pod is running as. securityContext.runAsUser

2. Preventing the container from running as root

   in case that an attacker push a dangerous image to the repository.

3. Running pods in privileged mode

   use protected system devices or other kernel features, which aren’t accessible to regular containers

   e.g listing all the devices of the node

4. Adding individual kernel capabilities to a container

   Fine-grained accessing modes are introduced (not only privileged and unprivileged) — kernel capabilities. Capabilities can be added or dropped from a container.

5. Preventing processes from writing to the container’s filesystem

   e.g preventing an attacker to modify the PHP file

All of the security constraint can be overridden by the specification of pods.

6. Allowing sharing volumes when containers run as different users:

   * fsGroup
   * supplementalGroup

   the fsGroup security context property is used when the process creates files in a volume


### Restricting the use of security-related features in pods

PodSecurityPolicy: cluster-level resources

If the pod comforms to the Policy, it can be created, otherwise it is discarded. 

See : 

https://github.com/luksa/kubernetes-in-action/blob/master/Chapter13/pod-security-policy.yaml

https://github.com/luksa/kubernetes-in-action/blob/master/Chapter13/psp-must-run-as.yaml



