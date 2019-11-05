# Securing Masters with Namespaces ![Best Practice: Security](https://img.shields.io/badge/best_practice-security-blue)

Provisioning Managed Masters into their own Kubernetes Namespace provides for more secure workloads by limiting Manage Master (and agents) access to Kubernetes `Secrets` and leveraging GKE Workload Identity for GCP IAM permissions.