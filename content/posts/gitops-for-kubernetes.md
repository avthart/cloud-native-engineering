---
title: "GitOps for Kubernetes"
date: 2019-01-01T15:00:59+02:00
draft: true
---

> Using Git as a single source of truth for declarative infrastructure and applications

This site uses [Hugo](https://gohugo.io/), [Netlify](https://www.netlify.com/), and [GitHub](https://github.com/avthart/cloud-native-engineering) and embraces Git as single source of thruth. The website is deployed automatically using a `git push`. The underlying infrastructure (Site hosting, DNS, HTTPS using Letsencrypt) is managed by Netlify. The official [Kubernetes website](https://github.com/kubernetes/website) uses the same stack.

This is just one example of using Git for declarative infrastructure and applications. 

Infrastructure-as-code, application configuration as code, deployments, monitoring, automated tests, etc. Developers and SysAdmins are more and more automating manual tasks by using new tooling and APIs to replace their manual processes.

## GitOps

Enter [GitOps](https://www.weave.works/technologies/gitops/). Initially proposed by Alexis Richardson from Weaveworks, GitOps is a way to do Continuous Delivery. It works by using Git as a single source of truth for declarative infrastructure and applications.  

{{< tweet 953638870888849408 >}}

GitOps describes best practices for faster app development while reducing risks:

* **Operations by pull requests** - everything is controlled through pull requests. Engineers already using Git for Continuous Integration. Make changes described desired state in Git, e.g. fix a production issue via a Pull Request instead of making changes to running system.
* **Everything is code** - having complete history configuration and state of your system in git (includes infrastructure, apps, config, monitoring). You will also get out-of-the-box auditing. 
* **Mean Time to Recovery** - simply rollback broken configuration using a `git revert` or redeploy a system in minutes if it failed completely.
* **Observability** - use diff tools to detect changes and get notifications in your favorite chat tool.
* **Security** - it is easier to commit the changes to a git repository, so users doest not have to manage hundreds of credentials. And you don 't need your CI system access to servers or a Kubernetes cluster. 

## Kubernetes

Kubernetes has become the defacto way of operating containers at scale on any public or private cloud. All major public cloud providers supports Kubernetes natively.

Most people are using `kubectl` directly for deploying and updating  applications.

{{< tweet 1070413458045202433 >}}

From a [security perspective and application delivery compliance](https://www.weave.works/blog/gitops-compliance-and-secure-cicd), only a few people should have access to the cluster using kubectl. From a developer perspective, everything that can go through a pipeline should go through a pipeline.

It is also a best practice to [separate config and source code repositories](https://github.com/argoproj/argo-cd/blob/master/docs/best_practices.md#separating-config-vs-source-code-repositories), so that there 's a clean history of config changes, access management in place (who can make changes, merge branches, etc). Besides that you can have one config repository (per team) and still have multiple source code repositories.

GitOps is a way to do Continuous Delivery. For Kubernetes this means using `git push` instead of `kubectl create/apply` or `helm install/upgrade`.

In order to apply GitOps for Kubernetes you need three things:

1. a Git repository with your workloads definitions in plain YAML/JSON format, Helm charts and any other Kubernetes custom resource that defines your cluster desired state
2. a container registry where your CI system pushes immutable images 
3. a Kubernetes controller that follows the operator pattern, your cluster always stays in sync with ‘the source of truth’ via its configuration files that are checked into Git. And since the desired state of your cluster is kept in Git, it can also be observed for differences against the running cluster.

There are a couple of interesting open source tools for Kubernetes that support GitOps best practices.

### Argo CD
[Argo CD](https://github.com/argoproj/argo-cd) follows the GitOps pattern of using git repositories as the source of truth for defining the desired application state. Kubernetes manifests can be specified in several ways:

* ksonnet applications
* kustomize applications
* helm charts
* Plain directory of YAML/json/jsonnet manifests

Argo CD automates the deployment of the desired application states in the specified target environments. Application deployments can track updates to branches, tags, or pinned to a specific version of manifests at a git commit.

[Getting started with ArgoCD](https://github.com/argoproj/argo-cd/blob/master/docs/getting_started.md)

### Jenkins X
[Jenkins X](https://jenkins-x.io/) is a CI/CD solution for modern cloud applications on Kubernetes. It provides fully automated and integrated CI/CD pipelines, environment promotion via GitOps, pull request preview environments and feedback on issues and pull requests. 

Jenkins X is using [Draft](https://draft.sh/) and [Helm](https://helm.sh/) to install both Jenkins X and to install the applications you create in each of the Environments (like Staging and Production)

[Getting started with Jenkins X](https://jenkins-x.io/getting-started/)

### Weave Flux
[Weave Flux](https://github.com/weaveworks/flux) is a tool that automatically ensures that the state of a cluster matches the config in git. It uses an operator in the cluster to trigger deployments inside Kubernetes, which means you don't need a separate CD tool. It monitors all relevant image repositories, detects new images, triggers deployments and updates the desired running configuration based on that (and a configurable policy).

[Getting started with Weave Flux](https://github.com/weaveworks/flux/blob/master/site/get-started.md)

## Future of GitOps

A lot has been written about GitOps as a “single source of truth for the whole system” for using Git for Kubernetes deployments on the cloud.

In one of the keynotes, CNCF COO, Alexis Richardson, presented the [CNCF 2020 vision](https://www.youtube.com/watch?v=qUK-F40oLVQ&t=752s). In the next couple of years will see a rise of “GitOps” as a new way of implementing Continous Delivery for Cloud Native applications.

I expect there will be more open source projects and vendors that will invest in GitOps.




