## Chapter 6 Summary

### Background — Volume

Processes are running inside a pod with shared resource like CPU, RAM, network interface and disk. File system of a container comes from the image and is defined during the build time of an image so that any change to the file system would be destroyed when the container is stopped. But under some scenarios, we want the new container to continue with what is left in the file system last time, therefore data must be persistent in the storage and this resource provided by Kubernetes is **volume**.

### Introduction

Volume is a component of a pod. (They can be shared among containers within a pod.)

Linux allows you to mount a filesystem at arbitrary locations in the file tree. Define a volume and let each container mount the volume at any location, in such they can share the files. The book illustrates this scenario with an example of a web service containing three container: a webserver container, a web content provider and a log rotater. The web content provider provides the web files for the webserver to run with. The webserver handles the requests and produces corresponding logs. The log rotater receives the logs. In this example, two volume or say two independent file systems are needed — files of web content and log files. 

### Usage:

#### File Sharing between Containers

1. EmptyDir: the volume starts out as an empty directory. It's useful to write tempory files for a single container and share files between multiple containers. 
2. GitDir: an empty directory with cloning from a Git repository — sidebar container. 

#### Accessing Files on the Worker Node's FS

HostPath: persistent storage of pods. 

Usage: Accessing the node's data. 

#### Persistent Storage

If the pod uses HostPath to persistently store data, then when the pod is rescheduled to another work node, the data is loss. This means the persistency requires the pod to be sensitive to the rescheduling. To decouple the scheduling of pods and persistency requirement, the persistent storage needs to take place on Network-attached-storage(e.g mongodb). 

NAS tools: GCE, AWS … 

### Decouple pods from underlying storage technology

The aim of Kubernets is to hide the actual infrastructure from the application and its developers, making the application portable across different cloud platforms and datacenters. 

Idea: Application requests storage resource from Kubernetes! 

1. PersistentVolume - provisioned by cluster admins. Infrastructure side
2. PersistentVolumeClaim - through which the pod consumes the storage.  Application side

PersistentVolume doesn't belong to any namespace. It's resource just like work nodes. 

Kubernetes cluster binds 1 and 2 when 2 is created. 

The policy of releasing persistent volume: retain(the volume can't be used by another pod), recycle, delete(the underlying volume is deleted)

### Dynamic provision of Persistent Volume

PersistentVolume provisioner: the admins define one or two (or more) StorageClasses and let the system create a new PersistentVolume each time one is requested through a PersistentVolumeClaim.



