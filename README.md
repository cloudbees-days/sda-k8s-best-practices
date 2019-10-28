# Best practices for CloudBees Core on Kubernetes Workshop

This workshop will provide a holistic set of best practices for running CloudBees Core on Kubernetes.

This repository contains instructions and learning materials for the workshop that is designed to teach the following key concepts:

  * General best practices for CI/CD on Kubernetes to include:
    * Scalability
    * Security
    * Performance
    * Resource Management
    * Configuration Management
  * How specific features of CloudBees Core on Kubernetes will accelerate your CD
    * Dynamic Master provisioning
    * Ephemeral and elastic Jenkins Kubernetes Agents
    * Master Configuration-as-Code with CloudBees' Configuration Bundles
  
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
  * A basic understanding of Docker: https://docs.docker.com/get-started/
  * A basic understanding of Kubernetes: https://kubernetes.io/docs/tutorials/kubernetes-basics/
  * A sub-domain to use for the workshop and the ability to add an A record to the DNS configuration for that domain
    >NOTE: If you don't have access to a domain that you can modify the DNS records then you can use [nip.io](https://nip.io).
   
Detailed setup instructions are available at [**Getting Started**](labs/getting-started.md)

# Workshop Labs

**The labs covered in this workshop are available at the following links:**

* Lab 1: [Getting Started](labs/getting-started/getting-started.md)
* Lab 2: [Installing CloudBees Core](labs/installing-core/installing-core.md)
* Lab 3: [Securing CloudBees Core with HTTPS and TLS](labs/tls/tls.md)
* Lab 4: [Resource Limits and Quotas](labs/limits-quotas/limits-quotas.md)
* Lab 5: [Managed Master Provisioning and Configuration](labs/managed-masters/managed-masters.md)
* Lab 6: [Kubernetes Agents](labs/k8s-agents/k8s-agents.md)
* Lab 7: [Pod Security Policies](labs/pod-security-policies/psp.md)


# Disclaimer

Although the examples and code in this repository was originally created by employees of CloudBees, Inc. to use in training customers, your use of this material is not sponsored or supported by CloudBees, Inc.

# Contributors 

* Contributors: [Kurt Madel](https://github.com/kmadel)
 
# Questions, Feedback, Pull Requests Etc.

If you have any questions, feedback, suggestions, etc. please submit them via issues or, even better, submit a Pull Request!


