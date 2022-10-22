# StaticPods
**StaticPods** konusuyla ilgili dosyalara buradan erişebilirsiniz.
***
The kubelet can manage a node independently.

We have the kubelet installed. We have Docker as well to run containers.

There is no Kubernetes cluster. So there are no Kube API server or anything like that.

The one thing that the kubelet knows to do is create PODs. But we don't have an API server here to provide POD details.  

How do you provide a pod definition file to the kubelet without a Kube-api server?

You can configure the kubelet to read the pod definitoon files from a directory on the server designated to store information about pods.

The kubelet periodically checks this directory for files, reads these files and creates pods on the host.  

So these PODs that are created by the kubelet on its own without the interbention from the API server or rest of the kubernetes cluster components are known as Static
PODs. We can only create pods this way. 


 
