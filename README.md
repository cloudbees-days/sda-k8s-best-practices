# Best practices for Software Delivery Automation on Kubernetes

This workshop will provide a holistic set of best practices for running CloudBees SDA on Kubernetes (SDA on K8s).

This repository contains instructions and learning materials for a workshop that is designed to teach the following key concepts:

  * General best practices for CI/CD on Kubernetes to include:
    * Scalability
    * Security
    * Performance
    * Resource Management
    * Configuration Management
  * How specific features of CloudBees SDA on Kubernetes will accelerate your CD
    * Dynamic Jenksin Controller provisioning
    * Ephemeral and elastic Jenkins Kubernetes Agents
    * Jenkins Controller Configuration-as-Code with CloudBees' Configuration Bundles
  
To get started goto the [**Getting Started**](labs/getting-started/getting-started.md).

# Workshop Prerequisites

In order to follow along with the hands on portion of the workshop students should have the following resources available to them:

  * Internet access to include access to https://github.com to include the ability to access and use [the GitHub File Editor](https://help.github.com/articles/editing-files-in-your-repository)
  * An account on https://github.com and a basic understanding of how to use GitHub to do things like fork a repository, edit files in the web UI, and create pull requests
  * A personal access token for your Github account ([Github-Personal-Access-Token.md](Github-Personal-Access-Token.md)) with the following permissions:
    - repo: all
    - admin:repo_hook: all
    - admin:org_hook
    - user: all
  * A Google Cloud Platform Account and Project
  * Understand what Kubernetes is: https://kubernetes.io/docs/concepts/overview/what-is-kubernetes/ 
    * [A basic understanding of Kubernetes](https://kubernetes.io/docs/tutorials/kubernetes-basics/)
    * [Containers vs Pods](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/)
    * [Persistent services with Stateful Sets](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/)
    * [Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/)
  * A sub-domain to use for the workshop and the ability to add an A record to the DNS configuration for that domain
    >NOTE: If you don't have access to a domain that you can modify the DNS records then you can use [nip.io](https://nip.io).
   
Detailed setup instructions are available at [**Getting Started**](labs/getting-started.md)

# Workshop Labs

**The labs covered in this workshop are available at the following links:**

* Lab 1: [Getting Started - Creating a GKE Cluster](labs/getting-started/getting-started.md)
* Lab 2: [Installing CloudBees CI](labs/installing-core/installing-core.md)
* Lab 3: [Securing CloudBees CI with HTTPS and TLS](labs/tls/tls.md)
* Lab 4: [Pod Security Policies](labs/pod-security-policies/psp.md)
* Lab 5: [Managed Master Provisioning and Configuration](labs/managed-masters/managed-masters.md)
* Lab 6: [Securing Masters with Namespaces](labs/master-namespaces/master-namespaces.md)
* Lab 7: [Hibernating Masters and Affinity](labs/hibernating-masters/hibernating-masters.md)
* Lab 8: [Configuration as Code for CloudBees CI](labs/casc-core/casc-core.md)
* Lab 9: [Resource Limits and Quotas](labs/limits-quotas/limits-quotas.md)
* Lab 10: [Kubernetes Agents](labs/k8s-agents/k8s-agents.md)
* Lab 11: [Pipeline Best Practices](labs/pipeline-best-practices/pipeline-best-practices.md)
* Lab 12: [Disaster Recovery for Core](labs/disaster-recovery/disaster-recovery.md)
* Lab 13: [Pod Network Policies]

# Disclaimer

Although the examples and code in this repository was originally created by employees of CloudBees, Inc. to use in training customers, your use of this material is not sponsored or supported by CloudBees, Inc.

# Contributors 

* Contributors: [Kurt Madel](https://github.com/kmadel)
 
# Questions, Feedback, Pull Requests Etc.

If you have any questions, feedback, suggestions, etc. please submit them via issues or, even better, submit a Pull Request!


