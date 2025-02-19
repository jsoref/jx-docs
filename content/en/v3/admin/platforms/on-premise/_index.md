---
title: On Premise
linktitle:  On Premise
type: docs
description: Setup Jenkins X on vanilla Kubernetes  
date: 2017-02-01
publishdate: 2017-02-01
lastmod: 2020-02-21
weight: 150
aliases:
  - /v3/admin/platform/on-premise
---

---
**NOTE**

Ensure you are logged into GitHub else you will get a 404 error when clicking the links below

---

## On Premise Kubernetes

If you are using kubernetes we highly recommend you use one of the [managed cloud providers](/v3/#administration) as this comes with lots of additional features like:

* container registries and bucket storage
* IAM and workload identity (e.g. so kubernetes Service Accounts can be assigned roles to be able to read/write to certain buckets or container registries) 

However sometimes you need to run kubernetes on your premise. Longer term we hope the cloud providers can run their managed kubernetes and associated infrastructure on your premise too so you get to reuse the same storage + IAM anywhere. But until then, this guide is intended to get you started installing Jenkins X on a vanilla kubernetes cluster on premise.

Here are some detailed [instructions](https://007ba7.us/howto/jx-install/) on getting JX running in a Lab Environment.

### Prerequisites

The following are the prerequisites of your on-premise kubernetes cluster:

* [Download and install the jx 3.x binary](/v3/guides/jx3/)

#### Kubernetes cluster

We obviously need a working kubernetes cluster. There are many approaches to [setting up on premise clusters](https://kubernetes.io/docs/setup/production-environment/tools/) obviously the easiest approach is to use the [cloud](/v3/#administration).

If you are going the bare metal route you could try [these instructions](https://007ba7.us/howto/k8s-install/)
 
#### kubectl access

You need to be able to connect to your kubernetes cluster via [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/) so that you can run commands like:

```bash 
kubectl get ns
kubectl get node
```

To view the namespaces and nodes respectively.
 
#### Ingress

To use Jenkins X we need ingress to work. This means being able to create a kubernetes `Ingress`  resource with a domain name which can be resolved outside of kubernetes to network into kubernetes services.

Jenkins X installs `nginx` which has a `LoadBalancer` kubernetes `Service` to implement ingress. But the underlying kubernetes platform needs to implement the load balancing network and infrastructure. This comes out of the box on all public clouds.
 
With an on-premise kubernetes cluster you need to install something like [MetalLB](https://metallb.universe.tf/)

If you are on bare metal you could try [these instructions](https://007ba7.us/howto/metallb/)

#### Storage

We need your kubernetes cluster to have a default [storage class](https://kubernetes.io/docs/concepts/storage/storage-classes/) so that `PersistentVolumeClaim` resources in helm charts get resolved to `PersistentVolume` resources so that persistent disks can be used.

You may find [these instructions useful](https://007ba7.us/howto/nfs-storage/)

### Getting Started

This is our current recommended quickstart for on premise kubernetes:

*  <a href="https://github.com/jx3-gitops-repositories/jx3-kubernetes/generate" target="github" class="btn bg-primary text-light">Create the cluster Git Repository</a> based on the [jx3-gitops-repositories/jx3-kubernetes](https://github.com/jx3-gitops-repositories/jx3-kubernetes/generate) template

    * if the above button does not work then please [Login to GitHub](https://github.com/login) first and then retry the button

* `git clone` the new repository via **HTTPS** and `cd` into the git clone directory

* find out what your ingress domain is for your cluster then modify the `jx-requirements.yml` file and modify the `ingress.domain` section...

```yaml
cluster:
...
ingress:
  domain: mydomain.com
...
```

* verify your cluster does not already have an [nginx](https://www.nginx.com/) installation. If it does then please remove the `nginx` line from your `helmfile.yaml` file and remove the `helmfiles/nginx` files. If you are using a custom nginx installation then you will need to figure out your domain by hand and won't be able to let Jenkins X detect the load balancer IP from its included nginx installation.

* git add, commit and push your changes:

```bash
git add *
git commit -a -m "fix: added domain"
git push origin main
```

* ensure you are connected to your cluster so you can run the following [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/) commands 

```bash 
kubectl get ns
kubectl get node      
```        

*  <a href="/v3/guides/operator/" 
    target="github" class="btn bg-primary text-light" 
    title="install the git operator to setup Jenkins X in your cluster">
    Install the git operator
  </a> from inside a git clone of the git repository you created above.

* switch to the `jx` namespace

```bash    
jx ns jx
```        

*  <a href="/v3/develop/create-project/" class="btn bg-primary text-light">Create or import projects</a> 


