---
title:       "K3s cluster architecture"
subtitle:    ""
description: ""
date:        2023-04-10
author:      "Afshin Yavari"
image:       ""
tags:        ["Kunermetes", "k3s", "architecture", "Gitops", "CI/CD"]
categories:  ["Kubernetes"]
---

# High level cluster architecture

This post will describe the k3s-cluster architecture, what components I will use and which steps I have to take before I can get ArgoCD up and running for Gitops. The goal is to do as few manual steps as possible until ArgoCD is installed and everything can be moved to git and a gitops approach.

![image info](/img/cluster-architecture.excalidraw-communication.png)

To be able to setup and access ArgoCD I need persistent storage. For simplicity I will setup an NFS-server on one of the worker nodes and then setup NFS CSI on each worker node connecting to the server, this I will achieve with help of the helm chart of [nfs-subdir-external-provisioner](https://github.com/kubernetes-sigs/nfs-subdir-external-provisioner).

My cluster will have 3 master nodes and I would like to avoid installing an external proxy such as nginx or haproxy on a virtual machine to proxy against the k8s API. Instead I will use [Kube-VIP](https://kube-vip.io/) which provides Kubernetes clusters with a virtual IP and load balancer for both the control plane (for building a highly-available cluster) and Kubernetes Services of type LoadBalancer without relying on any external hardware or software. I used to use MetalLB to provide services of type LoadBalancer but since I already will use Kube-VIP for the controle plane I might as well use it for LoadBalancing services as well.

For ingress I will use [ingress-nginx](https://github.com/kubernetes/ingress-nginx) which will let me access application routes from outside of the cluster, however the ingress needs an external ip so it will use Kube-VIP and a service of type LoadBalancer to get that external IP. This btw could also be a way to save cost if running on a cloud environment, so instead of having every service that should be exposed to the outside world having an external loadbalancer, one could use MetalLb or Kube-VIP together with an ingress controller so that only one or a few external loadbalancer would be needed.

I think this would be the minimum amount of software I would need to get ArgoCD up and running. Post ArgoCD installation I will install [cert-manager](https://cert-manager.io/) and the [Loopia-go](https://github.com/jonlil/loopia-go) webbhook integration for cert-manager so I can provision wildcard certificates to the ingress-controller from my Loopia domain.

To make things easy I will use Github and dockerhub to store code and images. But later on I will probably install either [Gitea](https://gitea.io) or [Gitlab](https://gitlab.com) together with [Harbor](https://goharbor.io/) container registry.

In the next post I will go through installation of the cluster. The starting point will be 6 virtual machines with Ubuntu-server. I will use [K3sup](https://github.com/alexellis/k3sup) to provision the k3s cluster, but more on that in the next post.