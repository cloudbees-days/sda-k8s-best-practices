# Best Practices for Pipelines with Core on Kubernetes


## Building and Pushing Container Images

Based on the Pod Security Policies we set up in a previous lab we are not able to build and push container images with Docker - at least not from a Kubernetes Pod based Jenkins agent.

## Binding IAM Permissions to Jenkins Kubernetes Agent Pods

Sometimes you need a Jenkins Pipeline that used different cloud services - such as pushing a container image to the Google Container Registry. Workload Identity for GKE allows us to bind GCP IAM Service Accounts (GSA) to Kubernetes Service Accounts (KSA) in a specific Kubernetes Namespace.
