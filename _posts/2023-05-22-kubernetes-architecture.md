---
layout: post
title: Kubernetes Architecture
categories: kubernetes
---

* [1 Architecture](#1-architecture)
    * [1.1 Kubernetes Objects](#1-1-kubernetes-objects)
    * [1.2 Kubernetes Components](#1-2-kubernetes-components)
    * [1.3 K8s API](#1-3-k8s-api)

## 1 Architecture

# 1 1 Kubernetes Objects

* They can describe the state of your deployed applications
* Resources available to those applications
* Policies around how those apps behave

# 1 2 Kubernetes Components

* Kubernetes Cluster
	* Consists of multiple worker nodes / physical machines
	* Each worker node consists of pods, which are containerized applications
	* The K8s Control Plane usually runs on multiple computers, when in a real production environment.
* Control Plane
	* Control plane components should ideally be deployed on machines dedicated to control plane functionality, and thus no application containers should be deployed here.
	* Control plane components:
		* Kube-apiserver
			* Manages API access to the Kubernetes control plane.
			* Can deploy multiple apiservers and load balance between them.
		* Etcd
			* Key-value store to store cluster data.
		* Kube-scheduler
			* When a pod gets created, the scheduler determines on which worker node it should run.
				* This decision is based on factors such as policies, resource requirements, locality etc.
		* Kube-controller-manager
			* Controls controller processes
			* Controller processes are logically separate; but in practice, to reduce complexity, they run as one single binary
			* Controller processes include:
				* Node controller: handling worker nodes
				* Job controller: handlings jobs
				* ServiceAccount controller: Manages service accounts when new namespaces are created for example, etc.
		* Cloud-controller-manager
			* It ensures only the necessary components in your cluster are talking to the cloud via cloud APIs, and that others are not.
			* Further, it consists of:
				* Node controller: Controlling nodes in the cloud, e.g. if one stops responding, checks if it's been deleted.
				* Route controller: Manage/create routes in the underlying cloud infra.
                * Service controller: manage/create load balancers in the cloud.

* Node Components:
	* Kubelet
		* Agent running on each worker node.
		* It creates Pods from PodSpecs
		* It only manages K8s-created containers/Pods
	* Kube-proxy
		* Network filtering
			* Either uses underlying OS firewalling capabilities or kube-proxy does network filtering itself
		* Accept/deny network traffic from within/outside the cluster
	* Container runtime
		* E.g. Docker.
* AddOns
	* DNS
		* DNS Server for the K8s Cluster. Pods have their own FQDN.
	* Web UI
		* Troubleshoot/management console for K8s cluster and/or the applications running in pods.
	* Cluster-level logging
		* Save container logs to a central log store

# 1 3 K8s API
* Kubectl
* Kubeadm (uses kubectl in the background)
* Can also use REST API calls
* Protobuf for intra-cluster communications
	* Protocol for retrieving cluster data from etcd key-value store => serializing => … => deserializing at client => usage
	* Apparently relatively fast
* K8s stores serialized objects in etcd (key-value store)
* API Changes
   * Access a resource via v1beta1 until the API is deprecated; then you can access via new API e.g. v1. But generally you can access them via both as long as they're both up.