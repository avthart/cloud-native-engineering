---
title: "10 Tips on the Certified Kubernetes Exam"
date: 2018-12-19T13:41:59+02:00
draft: false
---

I took the CKAD exam two weeks ago and passed it with 95% score at my first attempt! There 's a lot of overlap with the CKA exam, so I am planning to do this exam as well.

The exams are about speed and efficiency. For the CKA exam you will have 7,5 minute per question and for the CKAD exam you will only have 6 min per question. There 's no time to lookup the documentation for every question. You have to practice, practice and practice. 

With that said, time management is very important. Here are 10 tips on passing the exam.

### 1. Read exam resources

Take some time to read the exam resources (handbook, curriculum, exam tips and FAQ), so that you will know what is needed and what you can expect. 

* [Certified Kubernetes Administrator](https://www.cncf.io/certification/cka/)
* [Certified Kubernetes Application Developer](https://www.cncf.io/certification/ckad/)

### 2. Know how to quickly navigate the web and CLI documentation available to you

Familiarising yourself with the [official documentation](https://www.kubernetes.io/docs/) is a must for the exam. And most importantly, you can use the same documentation during the exam. 

You can the search function on the [Kubernetes](https://www.kubernetes.io/docs/) website.

While navigating the docs is important, the faster way is to use kubectl’s inbuilt documentation to quickly find information you need.

For figuring out which YAML field to use, `kubectl explain <object>.<subobject>` and `kubectl explain <object>.<subobject> --recursive`. They’ll print out types and attributes you need to set on a resource.

### 3. Do exactly what the question tells you to do.

This means that for example if it’s a POD they ask for, it’s a POD not a deployment with one POD.

### 4. Master the Linux Command Line
First of all, you will need to be comfortable navigating a Unix command line environment. The entire test takes place in a web application that simulates a Unix command line. You will need to be able to navigate a Unix shell, know a command-line text editor, and use Unix programs to complete questions.

The default editor for kubectl edit is VI. If you want to use nano, set the `KUBE_EDITOR` environment variable if you are not used to work with vi. 

```bash
export KUBE_EDITOR="nano"
```

### 5. Use kubectl auto completion

Load the kubectl completion code for bash into the current shell. This will speed up using kubectl on the command line.

```bash
source <(kubectl completion bash)
```

### 6. Use kubectl for creating resources
Use kubectl to create resources (such as deployment, pod, service, job, etc) instead of creating them from manifest files. It saves lot of time. If you need to make further changes, save the manifest file for the resource you created using kubectl, modify accordingly and re-apply.

The best way to approach the moderate to complex questions is to generate the initial YAML via the `--dry-run` flag. Then edit the file and create the required resource. 

```bash
kubectl run nginx --image=nginx --restart=Never --dry-run -o yaml > mypod.yaml
vi mypod.yaml
kubectl create -f mypod.yaml
pod "nginx" created
```

You can also extract YAML from a running resource with `--export` and `-o yaml`.

```bash
kubectl get deploy busybox --export -o yaml > exported.yaml
```

You can use the RUN command for Pods, Deployments, Jobs and CronJobs.

```bash
kubectl run nginx --image=nginx   (deployment)
kubectl run nginx --image=nginx --restart=Never   (pod)
kubectl run busybox --image=busybox --restart=OnFailure   (job)
kubectl run busybox --image=busybox --schedule="* * * * *"  --restart=OnFailure (cronJob)
```

I have found the following guide very useful for creating kubernetes resources using kubectl:
https://github.com/twajr/ckad-prep-notes#detailed-review

### 7. Spin up Kubernetes cluster (CKA exam only)

Spin up [Kubernetes The Hard Way](https://github.com/kelseyhightower/kubernetes-the-hard-way) by Keysel Hightower. This guide is optimized for learning, which means taking the long route to ensure you understand each task required to bootstrap a Kubernetes cluster.

Make sure to clean up the cluster once you are done with your practice as it incurs charges in your GCP account.

### 8. Make sure you understand the components that makes up the K8s cluster

How do they interact and what each component is responsible of, in order to be able to fix the issues for the troubleshooting tasks.

### 9. The exam runs on Ubuntu Linux 16

Make sure that you know how to interact with system services on this GNU Linux distribution.

### 10. You have a free retake

Don’t worry if you fail the exam—you get a free retake! 

Here is my CKAD certificate:
![CKAD](/img/CKAD_Certificate_Albert_sm.png "CKAD Certificate")