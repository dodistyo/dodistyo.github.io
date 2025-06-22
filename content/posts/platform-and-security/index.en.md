---
weight: 10
title: "Platform Engineering And Cyber Security Stories"
date: 2025-06-20T19:37:00+07:00
lastmod: 2025-06-20T19:37:00+07:00
draft: false
author: "Dodi"
authorLink: "https://github.com/dodistyo"
description: "My stories"
images: []
resources:
- name: "featured-image"
  src: "featured-image.png"
tags: ["security", "platform", "devops"]
categories: ["security", "platform"]
hiddenFromHomePage: false
comment:
  enable: true
---

Alright, so here are the takes from a guy who's been knee-deep in platform and security. building stuff, breaking stuff, securing it, then unbreaking it again. You know, the usual.
<!--more-->
## Connecting And Securing Hybrid Infrastructure {#connect-and-secure-infra}
Our world revolves around a hybrid infrastructure, a mix of on-premise data centers, multiple on-premise sites and cloud workloads in GCP. The first major hurdle was securing the traffic moving *between* these different environments. For connecting and securing this *north-south* traffic, we use `FortiGate` firewall and its SD-WAN capability for the solution, which effectively bridges all of our sites including cloud into one secure, private network.


To protect our public-facing applications, we picked `FortiWeb` as our WAF. We picked FortiWeb because we need a WAF solution that not only works for protecting our cloud applications, but also is able to protect our applications that are on-premises. We provision it in our cloud infrastructure using GCP PAYG licensing, so it’s more flexible in terms of scaling and budgeting. In order to make it reliable, we provision it in High-Availability mode with Active-Active instances. It sure doubled the cost, but in exchange for a more reliable system.


Then, we focused on securing the traffic *inside* the kubernetes clusters (east-west traffic). This required a two-part strategy: encryption and access control.


1. `Encryption`: To enforce a Zero Trust model, it was critical that all service-to-service communication was encrypted. For this, we use `Linkerd` (because of its light-weightness) as our service mesh, providing automatic, end-to-end encryption with mutual TLS (mTLS).

2. `Access Control`: While Linkerd encrypts the traffic, we still needed to control *which* services were allowed to talk to each other. For this granular, in-cluster firewalling, we use `Cilium` as our container network interface. Its ability to enforce network policies at both L4 and L7 gives us precise control over the traffic flow.

To further enhance security and automate operations, we built our own `security and operation service`. It acts as a lightweight, centralized API to handle common operational tasks. One of the feature functions is the integration with our runtime security tools, which is handled by `Falco`. When Falco detects a threat, it triggers our custom service to automatically isolate the compromised pod via a network policy. (Although not long after we built that, a new Falco component called `Falco Talon` emerged that could handle the proactive threat mitigation.)

With the infrastructure established, a new set of challenges emerged at the application layer. Managing secrets securely was a top priority. We have some options on the table, like using secret manager solutions from GCP or if we're fully into GitOps, we could use something like sealed secret/sops. For this, we are choosing HashiCorp `Vault` for the solution, providing a centralized and secure way to handle credentials. It's also easier to manage.



## Building Platform Ecosystem {#build-platform}
We're all in on Infrastructure as Code (IaC) as the foundation of how we manage things. Transforming all existing cloud infrastructure into code. We are picking `Terraform` just because it's everywhere in the GCP technical docs (Even though Terraform is no longer open source and we can technically use `OpenTofu` as an alternative). This ensures our infrastructure is reproducible, version-controlled, and managed with the same rigor as our application code.



Running workloads efficiently on Kubernetes also means managing the underlying nodes effectively. Manually scaling node pools and choosing machine types is inefficient. To solve this, we use GKE's `Node Auto-Provisioning (NAP)`. It acts much like Karpenter does for AWS but NAP is built-in GKE, automatically adding or removing node pools of different machine types based on the actual demands of our pending pods. This keeps our resource utilization high and our costs low without manual intervention.



Before applications even get to our GitOps, they go through our CI/CD process. We faced the challenge of running build pipelines efficiently without maintaining a fleet of static agents. For this, we use `Jenkins`, but with a modern approach. We leverage the Jenkins Kubernetes Operator, which dynamically provisions agents as pods inside our cluster. Now it's efficient enough, but to make this highly cost-effective, we ran these agent pods on GCP Spot VMs, scaling to zero when no pipelines are running. We’ve slashed build costs by up to 94% compared to always-on runners/agents. Yeah, Spot VMs are that good.



Next was application deployment. Deploying applications consistently across multiple Kubernetes clusters on the edge spanning our hybrid environment was a significant operational challenge. For this, we could use `ArgoCD` for the solution. It has been a lifesaver, seamlessly managing GitOps deployments to all clusters from a single source of truth. compared to FluxCD which lacks centralized visibility and will add additional system footprint.



Whatever my job title says, code keeps dragging me back in. Just like what I mentioned in the previous story, we develop custom services built with `Rust`. It’s not a really productive programming language to be honest, but the way it handles memory safety is kinda disciplining us to write better code. Besides, no garbage collector in rust so it’s more performant I believe. In order to make it even lighter, we built the service using `scratch` base image so it will run on bare bare minimum. It’s nice to use scratch as our base image, besides the small image it also helps us reduce the attack surface in the container level so it’s a plus for hardening our container security.



## Stay Relevant {#stay-relevant}
To deepen my understanding about how distributed systems work, I explore a bit about the `RAFT` consensus algorithm that powers `etcd` which is one of the main components of kubernetes. I even deepen my knowledge by rebuilding distributed redis by implementing another common distributed system technique called the consistent hashing of the hash ring. Reinventing the wheel is not bad after all.

Nowadays there is no way we don’t talk about AI, One of our recent projects involved is automatic code reviews. To solve this, we built a simple automation workflow using `n8n` and the `Gemini LLM` as the central brain to analyze code, flag issues. Even though n8n is not considered true agentic, it still works for our use cases and even we are confident enough to auto-merge simple MR changes.

It's been an incredibly rewarding year. By layering security from the perimeter down to the individual workloads and embracing automation at every level from IaC and node management to CI/CD and security response we've built a platform that is not just efficient, but resilient and secure by design.

### What’s next?
{{< typeit group=answer1 >}}
**More Cybersecurity and AI hands-on I guess..**
{{< /typeit >}}
{{< typeit group=answer1 >}}
**And hopefully more writing**
{{< /typeit >}}