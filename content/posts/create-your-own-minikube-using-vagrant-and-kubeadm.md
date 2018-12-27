---
title: "Create your own 'Minikube' using Vagrant and Kubeadm"
date: 2018-12-27T12:00:59+02:00
draft: false
---

I've been exploring severals options for my CKA exam preparation. It is very easy to start with [Minikube](https://github.com/kubernetes/minikube) for playing locally with Kubernetes and learning the basics. I've also followed [Kubernetes The Hard Way](https://github.com/kelseyhightower/kubernetes-the-hard-way) by Keysel Hightower. This guide is optimized for learning, which means taking the long route to ensure you understand each task required to bootstrap a Kubernetes cluster.

Also check my [exam tips](/posts/certified-kubernetes-exam-tips/) on getting certified.

##  Why not Minikube or Docker Mac?

You can easily use Minikube or run Kubernetes under Docker for Mac to run application code within Kubernetes, so both options are fine.

However, for my CKA exam preparation, I wanted to create my own local Kubernetes environment so that I understand how Kubernetes can be installed ("the easy way") and be able to tweak some parameters. Besides that I want to use standard tools like `kubeadm` that can be used for production environments.

## Install Virtualbox and Vagrant

With [Vagrant](http://www.vagrantup.com/) you can create and configure lightweight, reproducible, and portable development environments. Vagrant uses [Virtualbox](https://www.virtualbox.org/) to create virtual machines. 

First, let's install vagrant and virtualbox:

```bash
brew cask install virtualbox vagrant vagrant-manager
```

## Kubernetes Box

We are going to create a very simple Vagrant box:

* Guest VM with Ubuntu LTS 16.04 (used in CKA exam)
* Install Kubernetes with Kubeadm
* Run Kubernetes Control Plane and Worker as a single-node cluster

Let's create a basic box. We configure it with a DHCP network so that you can run `kubectl` from your local machine:

```bash
Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/xenial64"
  config.vm.network "private_network", type: "dhcp"
end
```

Create and ssh into this box:

```bash
vagrant up
vagrant ssh
```

## Install Kubernetes

Install Kubernetes packages (Docker, kubelet, kubeadm, kubectl) using the official repository:

```bash
sudo apt-get update && apt-get install -y apt-transport-https
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
sudo apt-get update
sudo apt-get install -y docker.io kubelet kubeadm kubectl
```

Kubelet requires swap off (after reboot):

```bash
swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```

Make sure kubelet will use the same cgroup driver as Docker:

```bash
sudo sed -i '0,/ExecStart=/s//Environment="KUBELET_EXTRA_ARGS=--cgroup-driver=cgroupfs"\n&/' /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
```

As mentioned before, we will use kubeadm to install the cluster. We only have to provide the node name and ip address that is known by externally, so that we don 't see SSL error messages when connecting from your local machine.

```bash
IPADDR=`ifconfig enp0s8 | grep Mask | awk '{print $2}'| cut -f2 -d:`
NODENAME=$(hostname -s)
sudo kubeadm init --apiserver-cert-extra-sans=$IPADDR  --node-name $NODENAME
```

To start using your cluster, you need to run the following as a regular user (= vagrant user):

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

Verify that cluster is working:

```bash
kubectl get componentstatus
NAME                 STATUS    MESSAGE              ERROR
scheduler            Healthy   ok
controller-manager   Healthy   ok
etcd-0               Healthy   {"health": "true"}
```

Get nodes:

```bash
kubectl get nodes
NAME            STATUS     ROLES    AGE   VERSION
ubuntu-xenial   NotReady   master   11m   v1.13.1
```

By default, you cannot run regular workloads on the master node. As we have only one node, we need to remove the master node role taint.

```bash
kubectl taint nodes --all node-role.kubernetes.io/master-
```

You should also deploy a pod network to the cluster, with one of the options listed at https://kubernetes.io/docs/concepts/cluster-administration/addons/

We will go for [Calico](https://docs.projectcalico.org/v3.4/getting-started/kubernetes/):

```bash
kubectl apply -f \
https://docs.projectcalico.org/v3.4/getting-started/kubernetes/installation/hosted/etcd.yaml
kubectl apply -f \ https://docs.projectcalico.org/v3.4/getting-started/kubernetes/installation/hosted/calico.yaml
```

Check if node status is `Ready`:

```bash
kubectl get nodes
NAME            STATUS   ROLES    AGE   VERSION
ubuntu-xenial   Ready    master   22m   v1.13.1
```

## Vagrantfile

I have create a Vagrantfile on GitHub with all steps so that you only have to run `vagrant up`:

{{< gist avthart d050b13cad9e5a991cdeae2bf43c2ab3 >}}

## Run Kubernetes Applications

Let's deploy a nginx container and get contents using POD IP address:

```bash
kubectl run nginx --image=nginx --port=80
kubectl get pods -o wide
curl $(kubectl get pods -l run=nginx -o jsonpath="{.items[0].status.podIP}")
```

Expose nginx deployment using a service: 

```bash
kubectl expose deployment nginx
kubectl get service
```

Get contents using DNS:

```bash
kubectl run curl --image=busybox -it --rm --restart=Never -- sh
wget -O- nginx.default.svc.cluster.local
```

Or using IP address:

```bash
curl $(kubectl get service -l run=nginx -o jsonpath="{.items[0].spec.clusterIP}")
```

## References

* https://www.vagrantup.com/docs/index.html
* https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/
* https://docs.projectcalico.org/v3.4/getting-started/kubernetes/