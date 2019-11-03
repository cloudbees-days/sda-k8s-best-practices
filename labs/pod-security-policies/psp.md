# Pod Security Policies ![Best Practice: Security](https://img.shields.io/badge/best_practice-security-blue)

[Pod Security Policies (PSPs)](https://kubernetes.io/docs/concepts/policy/pod-security-policy/) are built-in Kubernetes cluster-level resources that allow you to enforce security related properties of every container in your cluster. If a container in a Pod does not meet the criteria for an applicable PSP then it will not be scheduled to run and no Pod will be able to run at all if there is not a PSP applied to a `Role` or `ClusterRole` applied to the Kubernetes ServiceAccount used to start the Pod.

## Overview

Checkout [*Using Kubernetes Pod Security Policies with CloudBees Core - Jenkins*](https://technologists.dev/posts/best-practices-for-cloudbees-core-jenkins-on-kubernetes/core-psp/) for an in depth look at using PSPs with Core.

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
1. To get the `cjoc-0` Pod running again we will create a new `ClusterRole` and a `ClusterRoleBinding` to apply it to the `cjoc` `ServiceAccount`:
   1. Update the ***cb-restricted-psp.yml*** file with the following `ClusterRole`:
   ```yaml
   ---
   # restricted psp to be use accross cluster.
   apiVersion: rbac.authorization.k8s.io/v1
   kind: ClusterRole
   metadata:
     name: restricted-psp-cluster-role
   rules:
   - apiGroups:
     - policy
     resources:
     - podsecuritypolicies
     resourceNames:
     - cb-restricted
     verbs:
     - use
   ```
   1. Next we will add the following `RoleBinding` to the ***cb-restricted-psp.yml*** to bind the `restricted-psp-cluster-role` to the `cjoc` `ServiceAccount`:
   ```yaml
   ---
   apiVersion: rbac.authorization.k8s.io/v1
   kind: RoleBinding
   metadata:
     name: restricted-psp-role
   roleRef:
     apiGroup: rbac.authorization.k8s.io
     kind: ClusterRole
     name: restricted-psp-cluster-role
   subjects:
   - kind: ServiceAccount
     name: cjoc
     namespace: cb-core
   ```
   1. Update the ***kustomization.yml*** file to add ***cb-restricted-psp.yml*** as a `resource`:
   ```yaml
   namespace: cb-core
   images:
   - name: cloudbees/cloudbees-cloud-core-oc
     newTag: 2.190.2.2
   resources:
   - cloudbees-core.yml
   - cb-restricted-psp.yml
   patchesStrategicMerge:
   - set-ingress-host.yml
   - set-storageclass.yml
   ```
   And then apply it with `kubectl`:
   ```
   kubectl apply -k ./kustomize
   ```
2. The `cjoc-0` `Pod` will start shortly after applying the updated `kustomization.yml`. Verify that the `cjoc-0` Pod started successfully:
   ```
   kubectl -n cb-core describe sts cjoc
   ````
   You should see the following event:
   ```
   Normal   SuccessfulCreate  5s                     statefulset-controller\
     create Pod cjoc-0 in StatefulSet cjoc successful
   ```


## Lab Summary
In this lab we updated the cluster to use Pod Security Policies and created a restrictive policy to use with CloudBees Core Pods. In the [next lab](../managed-masters/managed-masters.md) we will look at provisioning Managed Masters from CloudBees Core Operations Center.
