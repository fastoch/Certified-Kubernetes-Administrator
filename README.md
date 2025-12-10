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

- **kube-apiserver**: the central hub and frontend of the Control Plane. All communication to and from the cluster goes through the API server.
  - it exposes a Kubernetes API, validates and processes all API requests, and coordinates all processes between control plane and worker nodes.
- **etcd**: the cluster's single source of truth; a consistent and highly-available distributed key-value store for all cluster data.
  - all cluster data includes its configuration, state, and metadata
  - for security and consistency, direct access to **etcd** is restricted. All interactions must go through the **kube-apiserver**.
- **kube-scheduler**: the matchmaker. Watches for new pods and assigns them to the best Node.
- kube-controller-manager: the autopilot. Runs various controller processes (Node controller, Replication controller, etc.) to maintain the desired state.


6/124
