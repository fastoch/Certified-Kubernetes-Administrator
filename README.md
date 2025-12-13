# Sources

- https://www.youtube.com/watch?v=Fr9GqFwl6NM (29 Oct 2025)

# About the exam

This course is designed for people who want to take the **CKA** examination.  
Which requires solving multiple hands-on tasks from a CLI within a 2-hour time limit.  
Candidates get two exam attempts (one free retake) and access to two Killer.sh simulators for practice.

# Why was Kubernetes invented?

In modern software development, applications are often packaged into lightweight portable units called containers.  
Managing hundreds or thousands of containers across a fleet of machines presents significant challenges.  

Kubernetes addresses these challenges by providing a robust framework for running distributed systems resiliently.  
Kubernetes is an open-source container orchestrator that automates the deployment, scaling, and management of containerized applications.  

>[!note]
>The "K8s" abbreviation for Kubernetes comes from "K + 8 letters + s". Kubernetes means "helmsman" or "pilot".

# Core Benefits of Kubernetes

## Self-healing

- automatically restarts failed containers
- replaces and reschedules containers when nodes die
- kills containers that fail health checks

## Automatic scaling

- scales applications up or down based on CPU, memory, or custom metrics

## Zero-Downtime Deployments

- facilitates rolling updates and rollbacks, ensuring service availability

# Core Philosophy: Declarative Configuration

- Kubernetes operates on a declarative model
- You define the desired state of the system in **YAML manifest files**
  - What applications should run
  - How many replicas
  - What network policies to enforce
  - and so on...

Once these manifests are applied to the cluster, Kubernetes controllers continuously work in a **"Control Loop"** to match the **actual state** to the **desired state**.  
- For example, if a pod crashes, a controller notices the discrepancy between the desired replica count and the actual count, and creates a new one

This model is the foundation of Kubernetes self-healing and automation capabilities.  

# Kubernetes Fundamentals & Lab Setup

## Understanding the Kubernetes Architecture

- Kubernetes follows a **control plane-worker node** architecture (which is a **master-slave** relationship)
- **Control Plane (The Brain)**: makes global decisions and manages the cluster's state
- **Worker Nodes (The Muscle)**: run the actual application workloads

### The Control Plane

The Control Plane is the set of components responsible for container orchestration and maintaining the desired state of the cluster.  
It can run on a single machine or be replicated across multiple machines for high-availability.  

- **kube-apiserver**: the central hub and frontend of the Control Plane.
  - All communication to and from the cluster goes through the API server.
  - it exposes a Kubernetes API, validates and processes all API requests, and coordinates all processes between control plane and worker nodes.
- **etcd**: the cluster's single source of truth
  - a consistent and highly-available distributed key-value store for all cluster data
  - all cluster data = its configuration, state, and metadata
  - for security and consistency, direct access to **etcd** is restricted. All interactions must go through the **kube-apiserver**.
- **kube-scheduler**: the cluster's matchmaker.
  - Watches for new pods that do not yet have a node assigned, and assigns them to the best node.
  - the node selection is based on a complex set of factors: resource requirements, hardware constraints, affinity and anti-affinity rules, and data locality
- **kube-controller-manager:** the cluster's autopilot.
  - Runs various controller processes (Node controller, Replication controller, etc.) to maintain the desired state
  - each controller runs a control loop that watches the shared state of the cluster (stored in **etcd**) through the kube-apiserver, and works to move the current state towards the desired state
 
### Worker Nodes (also called "workers")

They are the machines where your applications run (VMs or actual computers).  
They are managed by the Control Plane and contain all the necessary services to run containers.  

- **kubelet**: the primary "node agent", that runs on each worker node.
  - it communicates with the kube-apiserver to receive instructions and report the status of the node and its containers 
  - Ensures that the containers described in Pod specifications are running and healthy.
  - Manages the container lifecycle on its node.
- **kube-proxy**: a network proxy that runs on each node.
  - a fundamental part of the Kubernetes service concept
  - maintains network rules on the node to allow communication to your pods from both inside and outside the cluster
  - it can use various modes to direct traffic destined for a service's virtual IP to the correct backend pod
- **Container Runtime**: the software responsible for running the containers
  - Kubernetes supports multiple container runtime such as `containerd` and `CRI-O`
  - The kubelet communicates with the container runtime using the CRI (container runtime interface) to manage container operations like pulling images, and stopping or starting containers

## Core Kubernetes Objects

Kubernetes objects are persistent entities that represent your cluster's desired state.  
By creating an object, you are telling the kubernetes system what you want your cluster's workload to look like.  

### Pods 

- Pods are the smallest and most fundamental deployable unit in Kubernetes.  
- They represent a single instance of a running process in your cluster.  
- A pod encapsulates one or more tightly coupled containers that share storage and network resources 
- While a pod can contain multiple containers, the most common pattern is **one container per pod**

### ReplicaSets & Deployments

- **ReplicaSet**: ensures a specified number of identical Pods (replicas) are running at any given time.  
- **Deployment**: a higher-level object that manages ReplicaSets
  - it provides declarative updates to pods
  - handles rolling updates and rollbacks

You describe the desired state in a deployment, and the deployment controller changes the actual state to match the desired state at a controlled rate.  

Deployments are the standard and recommended way to manage stateless applications in Kubernetes.  

### Services

- Pods are ephemeral and their IP addresses change
- a service provides a stable endpoint to access a logical set of pods
- it acts as an abstraction, defining a policy by which to access the pods, and providing a stable virtual IP address (**ClusterIP**) and DNS name
- Traffic to a service is automatically load-balanced to the appropriate backend Pods

### Namespaces

- They provide a mechanism for isolating groups of resources within a single cluster.  
- They are a way to divide cluster resources between multiple users or teams
- Resource names must be unique within a namespace, but not across them

## Setting up your CKA practice environment

We will create a local cluster using **kubeadm**, the same tool used for many production clusters.  

### Prerequisites

Before installing Kubernetes, each machine or VM intended to be a node in the cluster must meet certain requirements:
- a compatible Linux host
- at least 2 GB of RAM per machine
- at least 2 CPUs for the control-plane node
- full network connectivity between all machines

### Step 1: installing and configuring a container runtime 

Kubernetes requires a container runtime on each node.  

While Docker was historically popular, the CKA exam environment and modern clusters typically use runtimes that directly implement a CRI (container runtime interface) 
such as **containerd** or **CRI-O**. We'll install **containerd**.  

The following commands must be run on all nodes (control plane + worker nodes) and require to use the Bash shell:

1. Let's ensure each node loads the kernel modules required for K8s:
```bash
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF
```
This command creates a config file which contains the specified Linux kernel modules:  overlay and br_netfilter.  
This file will be read by the system during boot, and these modules will be automatically loaded into the kernel every time the system starts.  

`cat` reads all subsequent lines as input until it encounters the EOF delimiter.  
`tee` copies stdin to both stdout and the specified file (k8s.conf)  
`overlay` enables overlay filesystem support for container storage, this is the preferred storage driver used by modern container runtimes.  
`br_netfilter` allows iptables filtering on bridged network traffic (K8s networking relies on the kernel's ability to see bridged traffic).

2. Then, let's activate these modules so we don't have to reboot right away:
```bash
sudo modprobe overlay
sudo modprobe br_netfilter
```

Now, Kubernetes components can start immediately and find the networking features they need.  

3. Next, let's make sure iptables correctly process bridged traffic, which is important for kube-proxy and the CNI plugin (container network interface):
```bash
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF
```
This creates another config file that will be read each time the system boots.  
The second line enables iptables rules on bridged traffic, which is crucial for Kubernetes pod networking.  
The next line does the same for IPv6.  
The last line activates IP forwarding for container-to-container routing.  

Then, we run the following command to activate these kernel settings without having to reboot the system:
```bash
sudo sysctl --system
```

4. Now we can finally install containerd (update your system first):
```bash
sudo dnf install containerd
```
The above command is for installing containerd on fedora, and will differ depending on your Linux system.

5. Now we can configure containerd. We'll generate the default config file, then modify it to use the system cgroup driver:
```bash
sudo mkdir -p /etc/containerd
sudo containerd config default | sudo tee /etc/containerd/config.toml
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml
```
The first line creates the directory that will host the containerd config file.  
The second line generates the default config file.  
The third line changes the setting `SystemdCgroup` from "false" to "true" in the containerd config file.  

>[!important]
>The kubelet and the container runtime must use the same cgroup driver to properly manage resource limits.  

Now, we just have to restart and enable containerd:
```bash
sudo systemctl restart containerd
sudo systemctl enable containerd
```

### Step 2: installing Kubernetes

Once again, in a real production cluster, the following needs to be done on all nodes.

>[!important]
>The kubelet component requires **swap** to be disabled to ensure predictable resource management and performance.

So let's disable swap and make this setting persistent across reboots:
```bash
sudo swapoff -a
sudo cp /etc/fstab /etc/fstab.bak
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```
The second line makes a backup of the fstab file, just in case, as errors could break the boot process.  
The third line is there to comment out swap entries in the /etc/fstab file.  

For disabling zram (Fedora's default), we need to create an empty config to override default settings: 
```bash
sudo touch /etc/systemd/zram-generator.conf
```

Now, I can install Kubernetes on my Fedora 43 node:
```batch
sudo dnf install kubectl kubelet kubeadm 
```
Fedora 43 includes Kubernetes packages directly in its standard repositories, so no external repo addition is typically needed.  

### Step 3: Preventing accidental upgrades

We need to prevent accidental upgrades, which is important for maintaining a stable cluster version.  
- For that, identify Kubernetes packages on your system (Fedora 43 in my case):
```bash
sudo dnf list --installed | grep kubernetes
```
- Then, lock the version for these packages with:
```bash
sudo dnf install dnf-plugins-core
sudo dnf versionlock add <Kubernetes_package_names>
```

You can check locked packages with `dnf versionlock list`.

### Step 4: Configuring a single-node cluster

For this practice environment, a single-node cluster is sufficient.  

1. First, we need to enable kubelet:
```bash
sudo systemctl enable kubelet
```

2. Then, we need to initialize our machine as a control plane:
```bash
sudo kubeadm init --pod-network-cidr=10.244.0.0/16
```

3. Next, we will configure kube-controller:
``` bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
The first line creates a directory dedicated to the kube-controller config.  
The second line copies the cluster administrator config file (created by kubeadm) to the standard location.  
The third line changes the file ownership so the current non-root user can securely interact with the cluster by running kubectl commands.  

4. By default, the control plane node is **tainted** to prevent regular application pods from being scheduled on it.  
But since we'll be running a single-node cluster, that taint must be removed via this command: 
```bash
kubectl taint nodes --all node-role.kubernetes.io/control-plane-
```
You'll get a confirmation message that your node has been untainted.  

5. The next command will install a CNI plugin, the Flannel one:
```bash
kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
```
The cluster is not fully functional until a Container Network Interface (CNI) plugin is installed.  
Without it, pods cannot communicate with each other, and coredns pods will not start correctly.  

After installing the CNI plugin, your node status should be "Ready", you can check that via `kubectl get nodes`.  

And you can run `kubectl get pods -A` to show all your 7 cluster's native pods, along with the flannel pod.  
Make sure all these pods are "running".  

---

>[!warning]
I got an issue with my coredns pods that were stuck at "ContainerCreating", while all other pods were running.  
I ran `ls /opt/cni/bin` and saw that there was no loopback plugin, only the flannel one.  
I installed all Linux CNI plugins via the following command:
```bash
curl -L "https://github.com/containernetworking/plugins/releases/download/v1.9.0/cni-plugins-linux-amd64-v1.9.0.tgz" | sudo tar -C /opt/cni/bin -xz`
```
Then I ran `kubectl get pods -A`, and this time all pods were running!  

The latest version of these CNI plugins can be found here: https://github.com/containernetworking/plugins/releases/ 

---

# Cluster Architecture, Installation & Configuration

We now have a functional single-node K8s cluster.  
This section will cover the core competencies of a K8s administrator: building, managing, and securing a multi-node cluster.  

In this section, we will:
- expand our single-node setup into a production-style, multi-node cluster,
- manage its lifecycle through upgrades and backups,
- implement high-availability,
- and configure access control and application management tooling.

## Bootstrapping a Multi-Node Cluster with `kubeadm`

Building a multi-node cluster involves a coordinated setup process across all machines.  
The following steps assume you have at least 2 nodes, one control plane, and one worker node.  

### Preparing all Nodes

The prerequisite steps must be completed on every node:
- ensure **unique** hostnames, MAC addresses, and product_uuids
- disable swap memory
- configure required kernel modules and sysctl settings (as done in the previous section)
- install a container runtime (containerd) with the systemd cgroup driver
- install kubeadm, kubelet, kubectl, and place them on hold

If your nodes are VMs (virtual machines), you need to edit their settings and make sure their IP and MAC addresses are different.  
You also need to set the network mode to "bridged" (in most cases, the default network mode is "shared").  

### 

We assume all nodes have been set up the same way we've set up our single-node cluster, except the control plane must remain tainted this time.  





23/124 
