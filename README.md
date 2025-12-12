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
 
### Worker Nodes

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

### First step: installing a container runtime

Kubernetes requires a container runtime on each node.  

While Docker was historically popular, the CKA exam environment and modern clusters typically use runtimes that directly implement a CRI (container runtime interface) 
such as containerd or CRI-O. We'll install **containerd**.  

The following commands must be run on all nodes (control plane and worker nodes):
- load required kernel modules: ``


12/124
