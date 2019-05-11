## Chapter 3 Summary

#### Pod

  a co-located group of containers and represents the basic block of Kubernetes. 

	1. higher abstract layer binds related processes together.
 	2. Achieve partial isolation by grouping containers with the same namespace so that they can share some resources but not all of them.
 	3. Volume is used by Kubernetes to enable file sharing. 
 	4. Separating processes to different pods: b.c they need different scaling strategies and to fully utilize hardware resources of infrastructure. 
      - Do they need to be run together or can they run on different hosts?
      - Do they represent a single whole or are they independent components?
      - Must they be scaled together or individually?

#### Pod with Labels

1. Specify labels when creating a pod: add label description in yaml file (—show-labels when listing pods)

2. modifying labels of existing pods: Since labels are just key-value pair description of a pod, we can change it whenever it is running or pending. 

   <u>Q: Modification on labels of existing pods is available in Kubernetes. So any lock machinism is involved to prevent any other kind of modification on running pods being affected while doing this? For example, the extermination of some pods with some labels.</u>

3. listing pods with specified labels — label selector. 

Pod Scheduling Policy:

The whole idea of Kubernetes is hiding the actual infrastructure from the apps that run on it. But in some cases, we need to have some control over the scheduling policy. We don't assign a particular pod to run an application but we can tell Kubernetes where the application should be deployed — If a certain node meets certain requirements, then this node can hold the application. 

Labels describe the traits of the working nodes, they tell whether the nodes meet the requirements. 

*Keywords: scheduling*

#### Pod with Annotations

Annotations are not used to group objects, in fact they are used to provide detailed information about the object. 

#### Namespace in Kubernetes

Group resources in unoverlapping groups with namespaces(not the namespace of Linux). E.g, separate the production environment, development enviornment and QA environment. 

```shell
kubectl create namespace custom-namespace
kubectl create -f kubia-manual.yaml -n custom-namespace
kubectl get pods --namespace custom-namespace
```

