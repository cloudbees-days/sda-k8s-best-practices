# Securing Masters with Namespaces ![Best Practice: Security](https://img.shields.io/badge/best_practice-security-blue)

Provisioning Managed Masters into their own Kubernetes Namespace provides for more secure workloads by limiting Manage Master (and agents) access to Kubernetes `Secrets` and leveraging GKE Workload Identity for GCP IAM permissions.

It also allows managing Kubernetes resources for individual Managed Master teams.

## Kubernetes Resources for Custom Managed Master Namespace

Before we can provision a Managed Master into its own `Namespace` we must create some `Roles` and `RoleBindings`  to allow Core OC to provision to this Master specific `Namespace` and to allow the Master specific `ServiceAccount` we are creating to provision Jenkins agents.

 1. 