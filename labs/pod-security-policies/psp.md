# Pod Security Policies ![Best Practice: Security](https://img.shields.io/badge/best_practice-security-blue)

[Pod Security Policies (PSPs)](https://kubernetes.io/docs/concepts/policy/pod-security-policy/) are built-in Kubernetes cluster-level resources that allow you to enforce security related properties of every container in your cluster. If a container in a Pod does not meet the criteria for an applicable PSP then it will not be scheduled to run and no Pod will be able to run at all if there is not a PSP applied to the Kubernetes ServiceAccount assigned to start the Pod.

## Overview


## Enable PodSecurityPolicies

1. We will use the `glcoud` CLI to enable PSPs for the Core GKE cluster:
2. Now we will restart the `cjoc-0` `Pod` to show that it will no longer start due to not having a PSP.
3. Create a restrictive PSP cluster resource in a new ***psp*** directory:
   ```
   mkdir psp
   ```

4. Apply the PSP to the `cjoc-master-management` `Role` resources by applying it to the `cjoc` `ServiceAccount` via a `ClusterRoleBinding`:
5. The `cjoc-0` `Pod` will now start:
6. Now we need to apply 