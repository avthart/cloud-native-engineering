---
title: "Cloud Native Journey"
date: 2018-09-25T14:18:46+02:00
draft: true
---

Cloud computing changed the required skills for software engineers and system administrators. A cloud-native approach requires new skills, technology and architectures to take advantage of the new world aka `the cloud`.

From the Cloud Native Computing Foundation:

> Cloud-native technologies empower organizations to build and run scalable applications in modern, dynamic environments such as public, private and hybrid clouds. Containers, service meshes, microservices, immutable infrastructure and declarative APIs exemplify this approach.

> These techniques enable loosely coupled systems that are resilient, manageble and observable. Combined with robust automation, they allow engineers to make high impact changes frequently and predictable with minimum toil.

## Cloud Native Journey

### Commitment

 It is very important that a cloud-native transformation needs to be defined as a strategic initiative with explicit support from executive management. This strategy demands significant upfront investment in infrastructure, platforms, people and skills.

 Technical leaders must challenge perceptions of the IT function as a cost centre by demonstrating the value that IT can bring to the business by transitioning to Cloud Native. A cloud-native approach means organizations can develop and deploy code much more frequently than before with reduced risks.
 
 Technical leaders also must evaluate which of their existing applications will benefit most from being rewritten as Cloud Native, and also which business initiatives and strategic priorities justify the investment of creating new cloud-native applications.  

### DevOps & Culture

Becoming a cloud-native company requires significant changes to how an IT department organises itself.

Cloud-native facilitates DevOps, a combination of people, process, automation and tools, resulting in a close collaboration between development and operations with the goal of constantly delivering high-quality software in production that solves customer challenges.

It has the potential to create a culture and an environment where building, testing and releasing software happens rapidly, frequently, and more consistently.

### Platform Team

![cloud native landscape](https://raw.githubusercontent.com/cncf/landscape/master/landscape/CloudNativeLandscape_latest.png "Cloud Native Landscape")


Above is a recent version of the [Cloud Native Landscape](https://landscape.cncf.io/): an attempt to capture some open source and commercial products grouped by arreas of concerns. This overview of (sometimes bleeding edge) technology, can be quite overwelming. Organisations need a platform which is build, automated and tested by a team which can support and facilitate feature teams with building and running systems, consisting of microservices or functions.

What can a cloud-native platform offer?
* The platform is self-service for the overwhelming majority of use cases.
* The platform is composable, containing discrete services that can be used independently.
* The platform does not force an inflexible way of working upon the delivery team
* The platform is quick and cheap to start using, with an easy on-ramp (e.g. quick start guides, documentation, code samples)
* The platform is secure and compliant by default
* The platform is up to date

Platform is a product, and needs a long-lived and stable product team tasked with both build and run. A platform should also be more than just software and APIs - it is documentation, and consulting, and support and evangelism, and templates and guidelines.

In most cases the best way to get started is to build a team that  build and operate a delivery infrastructure platform. The platform team ideally doesn’t even know what applications are running on the platform, they are only responsible for the availability of the platform services themselves.

In this way both application teams and platform teams have responsibility for build and run of their own products. `You build it, you run it` still applies.

### Software Architecture

### Packaging & Containerization

### CI/CD

### Infrastructure Automation

### Monitoring & Observability

### Deployment Patterns

Canary deployments
Blue-green deployments
GitOps

### Service Meshes

### Security Best Practices








## References

* https://martinfowler.com/articles/talk-about-platforms.html
* 