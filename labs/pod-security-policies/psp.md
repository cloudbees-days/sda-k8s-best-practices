# Pod Security Policies ![Best Practice: Security](https://img.shields.io/badge/best_practice-security-blue)

[Pod Security Policies (PSPs)](https://kubernetes.io/docs/concepts/policy/pod-security-policy/) are built-in Kubernetes cluster-level resources that allow you to enforce security related properties of every container in your cluster. If a container in a Pod does not meet the criteria for an applicable PSP then it will not be scheduled to run and no Pod will be able to run at all if there is not a PSP applied to a `Role` or `ClusterRole` applied to the Kubernetes ServiceAccount used to start the Pod.

## Overview


## Enable PodSecurityPolicies

1. We will use the [`glcoud` CLI to enable PSPs](https://cloud.google.com/kubernetes-engine/docs/how-to/pod-security-policies#enabling_podsecuritypolicy_controller) for the Core GKE cluster:
   ```
   gcloud beta container clusters update cb-core-workshop-cluster \
    --enable-pod-security-policy --region us-central1 --project k8s-core-best-practices
   ```
   This will take a few minutes as all the nodes (master and worker) must be recreated.
   >NOTE: Pod Security Policies is a beta feature for GKE.
2. Now we will try and restart the CloudBees Core Operations Center `Pod` by scaling the replicas of the **cjoc** `StatefulSet` to 0 and then back to 1:
   ```
   kubectl -n cb-core scale sts cjoc --replicas=0
   kubectl -n cb-core rollout status sts cjoc
   ```
   Once the rollout status goes to 0, update to set the replicas back to 1:
   ```
   kubectl -n cb-core scale sts cjoc --replicas=1
   ```
   Next describe the **cjoc-0** `Pod` to see why it won't start:
   ```
   kubectl -n cb-core describe sts cjoc
   ```
   You should see the following warning:
   ```
   Warning  FailedCreate      0s (x15 over 42s)  statefulset-controller\
     create Pod cjoc-0 in StatefulSet cjoc failed error: pods "cjoc-0" is forbidden:\
      unable to validate against any pod security policy: []
   ```
3. Create a restrictive PSP cluster resource in a file named ***cb-restricted-psp.yml*** in the ***kustomize*** directory with the following contents:
   ```yaml
   apiVersion: policy/v1beta1
   kind: PodSecurityPolicy
   metadata:
     name: cb-restricted
     annotations:
       seccomp.security.alpha.kubernetes.io/allowedProfileNames: 'runtime/default'
       apparmor.security.beta.kubernetes.io/allowedProfileNames: 'runtime/default'
       seccomp.security.alpha.kubernetes.io/defaultProfileName:  'runtime/default'
       apparmor.security.beta.kubernetes.io/defaultProfileName:  'runtime/default'
   spec:
     privileged: false
     fsGroup:
       rule: 'MustRunAs'
       ranges:
         - min: 1
           max: 65535
     runAsUser:
       rule: 'MustRunAs'
       ranges:
         - min: 1
           max: 65535
     seLinux:
       rule: RunAsAny
     supplementalGroups:
       rule: RunAsAny
     volumes:
     - 'emptyDir'
     - 'secret'
     - 'downwardAPI'
     - 'configMap'
     - 'persistentVolumeClaim'
     - 'projected'
     hostPID: false
     hostIPC: false
     hostNetwork: false
     allowPrivilegeEscalation: false
   ```
1. Apply the PSP to the `cjoc-master-management` `Role` resources by applying it to the `cjoc` `ServiceAccount` via a `ClusterRoleBinding`:
2. The `cjoc-0` `Pod` will now start:
3. Now we need to apply 