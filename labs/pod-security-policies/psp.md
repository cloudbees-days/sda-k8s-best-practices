![Best Practice: Security](https://img.shields.io/badge/best_practice-security-blue)
# Pod Security Policies

[Pod Security Policies (PSPs)](https://kubernetes.io/docs/concepts/policy/pod-security-policy/) are built-in Kubernetes cluster-level resources that allow you to enforce security related properties of every container in your cluster. If a container in a Pod does not meet the criteria for an applicable PSP then it will not be scheduled to run and no Pod will be able to run at all if there is not a PSP applied to the Kubernetes ServiceAccount assigned to start the Pod.

## Overview


## Enable PodSecurityPolicies

