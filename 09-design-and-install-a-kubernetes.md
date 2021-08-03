# Design and Install a Kubernetes Cluster

## Ask

* Purpose
  * Education
  * Development & Testing
  * Hosting Production Application
* Cloud or On Premise
* Workloads
  * How many?
  * What kind?
    * Web
    * Big Data/Analytics
  * Application Resource Requirements
    * CPU Intensive
    * Memory Intesive
  * Traffic
    * Heavy Traffic
    * Burst Traffic

## Purpose

* Education
  * Minikube
  * Single node cluster with kubeadm/CGP/AWS
* Development & Testing
  * Multi-node clsuter with a Single Master and Multiple workers
  * Setup using kubeadm tool or quick provision on GCP or AWS or AKS

## Hosting Production Applications

* High Availability Multi Node cluster with multiple master nodes
* Kubeadm or GCP or Kops on AWS or other supported platforms
* Upto 5000 nodes
* Upto 150000 PODs in the cluster
* Upto 300000 Total Containers
* Upto 100 PODs per Node

![Cluster Sizing](image/cluster-sizing.png)

## Cloud or OnPrem?

* Use kubeadm for on-prem
* GKE or GCP
* Kops for AWS
* Azure Kubernetes Services (AKS) for Azure

## Storage

* High Performance - SSD Backend Storage
* Multiple Concurrent connections - Network based storage
* Persistent shared volumes for shared access across multiple PODs
* Label nodes with specific disk types
* Use Node Selector to assign applications to node with specific disk types.

## Nodes

* Virtual or Physical Machines
* Minimum of 4 Node Cluster (Size based on workload)
* Master vs Worker Nodes
* Linux x86_64 Architecture

* Master nodes can host workloads
* Best practice is to not host workloads on Master nodes

## Master Nodes

* Etcd with three replicas
* API Server
* Controller
* Scheduler