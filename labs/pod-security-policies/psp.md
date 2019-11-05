# Pod Security Policies ![Best Practice: Security](https://img.shields.io/badge/best_practice-security-blue)

[Pod Security Policies (PSPs)](https://kubernetes.io/docs/concepts/policy/pod-security-policy/) are built-in Kubernetes cluster-level resources that allow you to enforce security related properties of every container in your cluster. If a container in a Pod does not meet the criteria for an applicable PSP then it will not be scheduled to run and no Pod will be able to run at all if there is not a PSP applied to a `Role` or `ClusterRole` applied to the Kubernetes ServiceAccount used to start the Pod.

## Overview

Checkout [*Using Kubernetes Pod Security Policies with CloudBees Core - Jenkins*](https://technologists.dev/posts/best-practices-for-cloudbees-core-jenkins-on-kubernetes/core-psp/) for an in depth look at using PSPs with Core.

## Enable PodSecurityPolicies

1. We will use the [`glcoud` CLI to enable PSPs](https://cloud.google.com/kubernetes-engine/docs/how-to/pod-security-policies#enabling_podsecuritypolicy_controller) for the Core GKE cluster:
   ```
   gcloud beta container clusters update  standard-cluster-1 \
    --enable-pod-security-policy --region us-central1 --project k8s-core-best-practices
   ```
   This will take a few minutes as all the nodes (master and worker) must be recreated. While the cluster is being updated let's take a look at [Pod Security Policies](https://kubernetes.io/docs/concepts/policy/pod-security-policy/).
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
   Some things to note about this `PodSecurityPolicy`:
   - `privileged`: set to `false` will disallow the use of Docker-in-Docker (DinD).
   - `runAsUser`: set to `MustRunAs` a range of **1 to 65535** so containers can’t run as the ROOT user.
   - `allowPrivilegeEscalation`: disable privilege escalation so that no child process of a container can gain more privileges than its parent.
   - `volumes`: Don’t allow mounting host directories/files as volumes by specifying specific volume types and not allowing the `hostPath` volume for any CD containers. This will disable the ability to [mount the Docker socket](https://github.com/jenkinsci/kubernetes-plugin/blob/master/examples/dood.groovy#L15).
4. To get the `cjoc-0` Pod running again we will create a new `ClusterRole` and a `ClusterRoleBinding` to apply it to the `cjoc` `ServiceAccount`:
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
5. The `cjoc-0` `Pod` will start shortly after applying the updated `kustomization.yml`. Verify that the `cjoc-0` Pod started successfully:
   ```
   kubectl -n cb-core describe sts cjoc
   ````
   You should see the following event:
   ```
   Normal   SuccessfulCreate  5s                     statefulset-controller\
     create Pod cjoc-0 in StatefulSet cjoc successful
   ```


## Pod Security Policies for Nginx Ingress and cert-manager

If we were to restart the Pods associated with ingress-nginx and cert-manager we would see that they would not start just as the `cjoc-0` Pod would not start above. Again, all Kubernetes `ServiceAccounts` must have a `Role`/`ClusterRole` with a valid PSP bound to them or else the `ServiceAccount` cannot be used to create a Pod.

### PSP for cert-manager and ingress-nginx

Because the **cert-manager** and **ingress-nginx** are both is their own `Namespace` we cannot just as their `ServiceAccounts` to the `restricted-psp-role` `RoleBinding` that we created for the `cjoc` `ServiceAccount`. We will need to create `RoleBindings` that allow those `ServiceAccounts` to use the `cb-restricted` PSP in their respective `Namespaces`. We will use a special Kubernetes `Group` to accomplish this - `system:serviceaccounts`.

1. Create a new file ***cert-manager-ingress-nginx-restricted-psp.yml*** file in the ***oc-casc** directory with the following `RoleBindings`:
   ```yaml
   ---
   # Bind the ClusterRole to the desired set of service accounts.
   # Policies should typically be bound to service accounts in a namespace.
   apiVersion: rbac.authorization.k8s.io/v1
   kind: RoleBinding
   metadata:
     name: cert-manager-psp-restricted
     namespace: cert-manager
   roleRef:
     apiGroup: rbac.authorization.k8s.io
     kind: ClusterRole
     name: restricted-psp-cluster-role 
   subjects:
   # Example: All service accounts in my-namespace
   - apiGroup: rbac.authorization.k8s.io
     kind: Group
     name: system:serviceaccounts
   ---
   apiVersion: policy/v1beta1
   kind: PodSecurityPolicy
   metadata:
     annotations:
       # Assumes apparmor available
       apparmor.security.beta.kubernetes.io/allowedProfileNames: 'runtime/default'
       apparmor.security.beta.kubernetes.io/defaultProfileName:  'runtime/default'
       seccomp.security.alpha.kubernetes.io/allowedProfileNames: 'docker/default'
       seccomp.security.alpha.kubernetes.io/defaultProfileName:  'docker/default'
     name: ingress-nginx
   spec:
     allowedCapabilities:
     - NET_BIND_SERVICE
     allowPrivilegeEscalation: true
     fsGroup:
       rule: 'MustRunAs'
       ranges:
       - min: 1
         max: 65535
     hostIPC: false
     hostNetwork: false
     hostPID: false
     privileged: false
     readOnlyRootFilesystem: false
     runAsUser:
       rule: 'MustRunAsNonRoot'
       ranges:
       - min: 33
         max: 65535
     seLinux:
       rule: 'RunAsAny'
     supplementalGroups:
       rule: 'MustRunAs'
       ranges:
       # Forbid adding the root group.
       - min: 1
         max: 65535
     volumes:
     - 'configMap'
     - 'downwardAPI'
     - 'emptyDir'
     - 'projected'
     - 'secret'
   ---
   apiVersion: rbac.authorization.k8s.io/v1
   kind: Role
   metadata:
     name: ingress-nginx-psp
     namespace: ingress-nginx
   rules:
   - apiGroups:
     - policy
     resourceNames:
     - ingress-nginx
     resources:
     - podsecuritypolicies
     verbs:
     - use
   ```
2. Apply the updates with `kubectl`:
   ```
   kubectl apply -f cert-manager-ingress-nginx-restricted-psp.yml
   ```
3. Now we will restart all the `Pods` in the `ingress-nginx` and `cert-manager` `Namespaces` and make sure they come back up. We will do that by deleting all the `Pods` in those two `Namespaces`:
   ```
   kubectl -n cert-manager delete pod --all 
   kubectl -n ingress-nginx delete pod --all 
   ```
4. Now let's see if the `Pods` were recreated in the the GCP console for **Kubernetes Engine** > **Workloads**.
5. 

## Lab Summary
In this lab we updated the cluster to use Pod Security Policies and created a restrictive policy to use with CloudBees Core Pods. In the [next lab](../managed-masters/managed-masters.md) we will look at provisioning Managed Masters from CloudBees Core Operations Center.
