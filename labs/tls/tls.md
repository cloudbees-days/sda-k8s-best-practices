# HTTPS for CloudBees Core

[cert-manager](https://docs.cert-manager.io/en/latest/index.html) and LetsEncrypt for TLS termination at ingress to allow us to use the HTTPS protocol for CloudBees Core.

## Install cert-manager

1. In the Cloud Shell code editor navigate to the ***oc-casc*** directory, create a new directory named ***cert-manager*** and in that directory create a new file named ***cert-manager-namespace.yml** with the following contents:
   ```yaml
   apiVersion: v1
   kind: Namespace
   metadata:
     name: cert-manager
     labels:
       app.kubernetes.io/name: cert-manager
   ```
2. Next use `kubectl` to apply that file:
   ```
   cd cert-manager
   kubectl apply -f cert-manager-namespace.yml
   ```
3. Now download the cert-manager single yaml manifest file containing the Kubernetes resources for cert-manager and apply:
   ```
   wget https://github.com/jetstack/cert-manager/releases/download/v0.9.0/cert-manager.yaml
   kubectl apply -f cert-manager.yaml
   ```
   >NOTE: There is a validation issue with the cert-manager CRDs in v0.10.0 and v0.11.0
4. 