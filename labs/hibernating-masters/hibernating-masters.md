# Managed Masters Hibernation

CloudBees Core has a feature that allows you to configure Managed Masters to **hibernate** after a specified amount of time if there are no active or scheduled builds. When used along with the Kubernetes cluster-autoscaler this feature will allow you to reduce costs by automatic scaling down of Kubernetes worker nodes when Managed Masters hibernate.

## Enabling Hibernation

Managed Master hibernation for CloudBees Core consists of two components:

1. A `managed-master-hibernation-monitor` service developed with [Quarkus](https://quarkus.io/) to be deployed to the same `Namespace` as CloudBees Core. It offers APIs to interact with a Managed Master (MM) which may be hibernated. [MM] in URIs refers to the name of a master. [REST-OF-PATH] can be any path suffix, perhaps empty, and may include a query string.
2. The CloudBees Managed Master Hibernation Plugin 

### Install `managed-master-hibernation-monitor` `Service`

1. Download the `managed-master-hibernation-monitor` Kubernetes YAML manifest into the ***kustomize*** directory:
2. Update the ***cb-restricted-psp.yml*** file to add the `managed-master-hibernation-monitor` `ServiceAccount` as a `subject` to the `restricted-psp-role` `RoleBinding`:
3. Apply 

### Install and Configure the CloudBees Managed Master Hibernation Plugin

1. 

## Node and Pod Affinity for Hibernation

A balance between cost savings and high availability.