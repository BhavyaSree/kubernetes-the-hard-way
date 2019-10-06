# Kubernetes Components (v1.16)
* Master Components
* Node Components


## Master Components
Control plain of the cluster, responsible for global decisions about cluster. 
* kube-apiserver
* etcd
* kube-scheduler
* kube-controller-manager
* cloud-controller-manager

### kube-apiserver
`kube-apiserver` is the only door and is the front-end for the Kubernetes control plane, it exposes the Kubernetes API.

### etcd
`etcd` is a distributed reliable, highly-available key-value store. The state of the Kubernetes resides in etcd. 

### kube-scheduler
`kube-scheduler` watches newly created pods that have no node assigned, and selects a node for them to run on. Scheduling decisions depends on individual and collective resource requirements, hardware/software/policy constraints, affinity and anti-affinity specifications, data locality, inter-workload interference and deadlines.

### kube-controller-manager
Kubernetes controller-manager is the combination of following submodule daemons. 
* Node Controller
* Replication Controller
* Endpoints Controller
* Service Account & Token Controllers

These controllers watches the state of the cluster through `apiserver` and make changes attempting to move current state towards desired state.

#### Node Controller
Node Controller manages various aspects of nodes. 
* Assigns a CIDR block to the node when it is registered. 
* Works with cloud provider to maintain the node list upto date.
* Moniters nodes' health. Sets the NodeStatus from to NodeReady to ConditionUnknown.
* Responsible for evicting pods that are running on nodes with `NoExecute` taints.

#### Replication Controller
Responsible for maintaining the correct number of pods for every replication controller object in the system.

#### Endpoints Controller
Populates the Endpoints object (that is, joins Services & Pods).

#### Service Account & Token Controllers
Create default accounts and API access tokens for new namespaces.

### cloud-controller-manager
Kubernetes Cloud Controller Manager runs controllers that interact with the underlying cloud providers. 

## Node Components
Node components run on every node, maintaining running pods and providing the Kubernetes runtime environment.

* kubelet
* kube-proxy
* Container Runtime

### kubelet
`kubelet` runs on every node in the cluster. It makes sure that containers are running in a pod.
The kubelet takes a set of PodSpecs that are provided through various mechanisms and ensures that the containers described in those PodSpecs are running and healthy. The kubelet doesn’t manage containers which were not created by Kubernetes.

### kube-proxy
`kube-proxy` is a network proxy that runs on each node in the cluster, implementing part of the Kubernetes `Service` concept.

kube-proxy maintains network rules on nodes. These network rules allow network communication to your Pods from network sessions inside or outside of your cluster.

kube-proxy uses the operating system packet filtering layer if there is one and it’s available. Otherwise, kube-proxy forwards the traffic itself.

### Container Runtime
The container runtime is the software that is responsible for running containers.

Kubernetes supports several container runtimes: Docker, containerd, cri-o, rktlet and any implementation of the Kubernetes CRI (`Container Runtime Interface`).
