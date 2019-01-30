---
title: "Multiple Kubernetes Clusters with Rancher"
date: 2019-01-30T15:05:59+02:00
tags: [kubernetes]
draft: false
---

After you've decided to run your application on Kubernetes, you are wondering how to organize your teams to do development on Kubernetes. This involves many opinions and strategies.

There are a couple of Kubernetes provides features which you can use for running multiple applications on one single cluster:

* Namespaces
* Roll Bases Access Control
* Network Policy
* Resource constraints and quotas

You can read more about these features in [Optimizing Cluster Resources for Kubernetes Team Development from Weaveworks](https://www.weave.works/blog/optimizing-cluster-resources-for-kubernetes-team-development)

Most organizations will create one cluster for testing, another one for staging or acceptance and then a third cluster for production. Sometimes organizations also want clusters for their development or preview environments. Besides that some organizations wants to run Kubernetes workloads in multiple clouds or in their private cloud.

You can end up with a lot of clusters, sometimes in different clouds. Multiple clusters adds a lot more overhead, maintenance and costs.

In the area of 'managing multiple Kubernetes clusters' there are some interesting tools and platforms which can help you removing some overhead, maintenance but also and adding cluster insights. 

In this post we will deploy Rancher on AWS EKS.

## Rancher

[Rancher](https://rancher.com/what-is-rancher/overview/) is a platform for managing multiple Kubernetes clusters. Rancher users have the choice of creating Kubernetes clusters with Rancher Kubernetes Engine (RKE) or cloud Kubernetes services, such as GKE, AKS, and EKS. Rancher users can also import and manage their existing Kubernetes clusters created using any Kubernetes distribution or installer.

## Prepare

You will need an AWS account, domain name and to setup the following tools for this guide.

Install and configure AWS cli

```
brew install awscli
```

```
aws configure
AWS Access Key ID [None]: ****
AWS Secret Access Key [None]: ****
Default region name [None]: us-west-2
Default output format [None]:
```

Install kubernetes-client

```
brew install kubernetes-cli
```

Install eksctl

```
brew install weaveworks/tap/eksctl
```

Now we are ready to create an EKS cluster which will be used to install Rancher.

## Create EKS cluster

We create a EKS cluster using eksctl. [eksctl](https://eksctl.io/) is a simple CLI tool for creating clusters on EKS

```
eksctl create cluster --name=rancher-management --nodes=3
```

> Launching EKS and all the dependencies will take approximately 15 minutes

Once you have created a cluster, you will find that cluster credentials were added in ~/.kube/config.

Verify that EKS cluster is created

```
eksctl get cluster rancher-management
```

Get Kubernetes cluster-info and nodes

```
kubectl cluster-info
kubectl get nodes
```

## Initialize Helm

[Helm](https://helm.sh) helps you manage Kubernetes applications. Helm Charts helps you define, install, and upgrade even the most complex Kubernetes application.

First, install helm

```
brew install kubernetes-helm
```

When you have a Kubernetes cluster with RBAC. You first need a ServiceAccount for Tiller (server component of Helm which will be installed within your cluster).

```
cat <<EOF | kubectl create -f -
apiVersion: v1
kind: ServiceAccount
metadata:
  name: tiller
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: tiller
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
  - kind: ServiceAccount
    name: tiller
    namespace: kube-system
EOF
```

Initialize helm client and server with:

```
helm init --service-account tiller
```

> NOTE: Read more about securing Helm and Tiller here https://docs.helm.sh/using_helm/#securing-your-helm-installation

## Install Rancher

Rancher installation is managed using Helm.

Add Helm Chart Repository

```
helm repo add rancher-latest https://releases.rancher.com/server-charts/latest
```

> NOTE: You can replace 'latest' with 'stable'. [Choosing a Version of Rancher](https://rancher.com/docs/rancher/v2.x/en/installation/server-tags/#helm-chart-repositories)

Rancher relies on [cert-manager](https://github.com/kubernetes/charts/tree/master/stable/cert-manager) version v0.5.2 from the official Kubernetes Helm chart repository to issue certificates from Rancher’s own generated CA or to request Let’s Encrypt certificates.

Install cert-manager from Kubernetes Helm chart repository.

```
helm install stable/cert-manager \
  --name cert-manager \
  --namespace kube-system \
  --version v0.5.2
```

Rancher is secure by default and will generate a self-signed certificate. You can also provide [your own or use Let's Encrypt](https://rancher.com/docs/rancher/v2.x/en/installation/ha/helm-rancher/#choose-your-ssl-configuration).

Wait for cert-manager to be rolled out:

```
kubectl -n kube-system rollout status deploy/cert-manager

Waiting for deployment "cert-manager" rollout to finish: 0 of 1 updated replicas are available...
deployment "cert-manager" successfully rolled out
```

We also need an nginx ingress controller and AWS for L4 load-balancing:

```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/mandatory.yaml
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/provider/aws/service-l4.yaml
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/provider/aws/patch-configmap-l4.yaml
kubectl patch service ingress-nginx -p '{"spec":{"externalTrafficPolicy":"Local"}}' -n ingress-nginx
```

Install Rancher

```
helm install rancher-latest/rancher \
  --name rancher \
  --namespace cattle-system \
  --set hostname=rancher.dock-yard.nl
```

Wait for rancher to be rolled out:

```
kubectl -n cattle-system rollout status deploy/rancher

Waiting for deployment "rancher" rollout to finish: 1 of 3 updated replicas are available...
Waiting for deployment "rancher" rollout to finish: 2 of 3 updated replicas are available...
deployment "rancher" successfully rolled out
```

## Using Rancher

Open the Rancher UI in your browser (see hostname above). You can see that your EKS cluster (local) will be imported.

You are now able to create new Kubernetes clusters on AWS EKS, Azure AKS, Google GCP, VMware vSphere and custom (e.g. bare metal). You can also import existing Kubernetes clusters.

Alternatively, you can also use the [Rancher CLI](https://rancher.com/docs/rancher/v2.x/en/cli/)

```
brew install rancher-cli
```

Login using an [API token](https://rancher.com/docs/rancher/v2.x/en/user-settings/api-keys/):

```
rancher login --name rancher-management --token token-**** https://rancher.dock-yard.nl/v3
```

List clusters

```
rancher clusters
CURRENT   ID        STATE     NAME      PROVIDER   NODES     CPU       RAM             PODS
*         local     active    local     Imported   3         0.53/6    0.14/22.22 GB   18/87
```

List nodes

```
rancher nodes
ID                    NAME                                           STATE     POOL      DESCRIPTION
local:machine-6qfsl   ip-192-168-28-79.us-west-2.compute.internal    active
local:machine-lqwzj   ip-192-168-51-182.us-west-2.compute.internal   active
local:machine-ngxsr   ip-192-168-65-239.us-west-2.compute.internal   active
```

You can do `rancher help` as this will show a list of available commands.

## Alternatives

In this guide we have played with Rancher, but there are some interesting alternatives out there:

* [Docker EE](https://docs.docker.com/ee/)
* [Gardener](https://gardener.cloud/)
* [Giant Swarm](https://giantswarm.io/)
* [Kubermatic](https://kubermatic.io/)
* [Mesosphere Kubernetes Engine](https://mesosphere.com/product/kubernetes-engine/)
* [Pivotal Container Service](https://pivotal.io/platform/pivotal-container-service)

Cloud managed Kubernetes services like AKS, EKS and GKE can be created with one single click or command-line (`terraform apply`). Kubernetes cluster mangement tools come with some extra overhead, management and costs. Try to find out if you really need one and which one is the best match for your use case.