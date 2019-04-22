# Chapter 2 Summary

https://books.apple.com/us/book/kubernetes-in-action/id1437029204

1. Pull and run any publicly available container image. Package your apps into container images and make them available to anyone by pushing the images to a remote image registry

   We can download the image from the  [registry](http://docker.io)and deploy the image everywhere. Pull is hidden by docker and triggered when run is called by image is not downloaded locally.

   `docker run busybox echo "Hello world"` <u>Page 130</u> 

   How to create an application image and run the image? 

   * Create an application: code, resource … 

   * Create a Dockerfile for the image: define the image layers — what is the image "dependency", add the application into the image and define the entry point of this image.

   * Build the container image: `docker build -t <image name>` “

     > The build process isn’t performed by the Docker client. Instead, the contents of the whole directory are uploaded to the Docker daemon and the image is built there.” P144

   * Run the container image: `docker run --name <container_name> -p <local port>:<container port> -d <image name>`

   * Stop the container: `docker stop <container_name>`

   * Push the image to an image registry: attach a tag to the image then push.

   ----

2. Enter a running container and inspect its environment

   Set up a multi-node Kubernetes cluster on Google Kubernetes Engine

   Configure an alias and tab completion for the kubectl command-line too

   List and inspect Nodes, Pods, Services, and ReplicationControllers in a Kubernetes cluster

   Run a container in Kubernetes and make it accessible from outside the cluster

   Tools: Kubernetes, Minikube(single-node mode), Google Kubernetes Engine, kubectl command line

   ---

3. Have a basic sense of how Pods, ReplicationControllers, and Services relate to one another

   ![View on pods, rc and service](/Users/kuangwanjing/Documents/Books/ReadingNotes/KubernetesInAction/ch2_1.jpg)

   * Pods are the basic deployment unit which may contain one or more containers. They look like an independent space with individual interal ip address and port number.

   * RC manages the lifetime of the pods. How to instruct the RC to do its work —  **declarative** but not commandative: eg. 

     `kubectl scale rc kubia --replicas=3`

   * Service is a static entry point. 

     <u>Question</u>: How Service know the living pods and how do service and rc maintain consistent about pods?

   

4. Scale an app horizontally by changing the ReplicationController’s replica count

5. Access the web-based Kubernetes dashboard on both Minikube and GKE

