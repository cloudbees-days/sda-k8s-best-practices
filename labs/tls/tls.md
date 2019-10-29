# HTTPS for CloudBees Core

[cert-manager](https://docs.cert-manager.io/en/latest/index.html) and LetsEncrypt for TLS termination at the Kubernetes cluster ingress to allow us to use the HTTPS protocol for CloudBees Core.

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
   kubectl apply -f ./cert-manager/cert-manager-namespace.yml
   ```
3. Now download the [cert-manager single yaml manifest file](https://docs.cert-manager.io/en/latest/getting-started/install/kubernetes.html#installing-with-regular-manifests) containing the Kubernetes resources for cert-manager and apply:
   ```
   wget https://github.com/jetstack/cert-manager/releases/download/v0.11.0/cert-manager.yaml
   kubectl apply --validate=false -f ./cert-manager/cert-manager.yaml
   ```

4. Verify that all of the cert-manager components are running:
   ```
   kubectl -n cert-manager get all
   ```
   That command should return the following:
   ```
   NAME                                          READY   STATUS    RESTARTS   AGE
   pod/cert-manager-69568fb89d-w865s             1/1     Running   1          26d
   pod/cert-manager-cainjector-6885996d5-6ggb8   1/1     Running   6          26d
   pod/cert-manager-webhook-59dfddccfd-z8ml9     1/1     Running   0          26d

   NAME                           TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
   service/cert-manager-webhook   ClusterIP   10.11.240.114   <none>        443/TCP   169d

   NAME                                      READY   UP-TO-DATE   AVAILABLE   AGE
   deployment.apps/cert-manager              1/1     1            1           169d
   deployment.apps/cert-manager-cainjector   1/1     1            1           169d
   deployment.apps/cert-manager-webhook      1/1     1            1           169d

   NAME                                                DESIRED   CURRENT   READY   AGE
   replicaset.apps/cert-manager-69568fb89d             1         1         1       169d
   replicaset.apps/cert-manager-cainjector-6885996d5   1         1         1       169d
   replicaset.apps/cert-manager-webhook-59dfddccfd     1         1         1       169d
   ```
5. Now we will create a staging cert-manager Issuer that will dynamically - a cert-manager CRD that is configured to dynamically issue certs for properly configured Ingresses. The Issuer that we are creating will obtain signed [x509 certificates](https://en.wikipedia.org/wiki/X.509) from [Let's Encrypt](https://letsencrypt.org/) (a free and automated Certificate Authority) using the [Automated Certificate Management Environment (ACME) protocol](https://en.wikipedia.org/wiki/Automated_Certificate_Management_Environment). In the Cloud Shell code editor create a file named ***letsencrypt-staging-issuer.yml*** in the ***cert-manager*** directory with the following contents, replacing the email value with your own: 
   ```yaml
   apiVersion: cert-manager.io/v1alpha2
   kind: Issuer
   metadata:
     name: letsencrypt-staging
     namespace: cb-core
   spec:
     acme:
       # You must replace this email address with your own.
       # Let's Encrypt will use this to contact you about expiring
       # certificates, and issues related to your account.
       email: user@example.com
       server: https://acme-staging-v02.api.letsencrypt.org/directory
       privateKeySecretRef:
         # Secret resource used to store the account's private key.
         name: letsencrypt-staging-issuer-account-key
       # Add a single challenge solver, HTTP01 using nginx
       solvers:
       - http01:
           ingress:
             class: nginx
   ```
   There is only one `solvers` and this is used to verify domain ownership. Let’s Encrypt gives a token to the cert-manager ACME client, then the cert-manager ACME client creates a `Solver` Pod with a file available at http://<YOUR_DOMAIN>/.well-known/acme-challenge/<TOKEN>. That file contains the token, plus a thumbprint of your account key. Once the cert-manager ACME client tells Let’s Encrypt that the file is ready, Let’s Encrypt tries retrieving it and when successful issues a certificate.
6. Next, create the cert-manager `Issuer` resource using `kubectl apply`:
   ```
   kubectl apply -f ./cert-manager/letsencrypt-staging-issuer.yml
   ```
   And then verify that the account was registered properly using `kubectl describe`:
   ```
   kubectl -n cb-core describe issuer letsencrypt-staging
   ```
7. Update the nginx-ingress for CloudBees Core to use TLS and use the staging cert-manager `Issuer` by modifying the ***set-ingress-host.yml*** file in the ***kustomize*** directory so it matches the following:
   ```yaml
   apiVersion: extensions/v1beta1
   kind: Ingress
   metadata:
     name: cjoc
     annotations:
       kubernetes.io/ingress.class: nginx
       cert-manager.io/issuer: "letsencrypt-staging"
       nginx.ingress.kubernetes.io/app-root: "http://$best_http_host/cjoc/teams-check/"
       nginx.ingress.kubernetes.io/ssl-redirect: "true"
       # "413 Request Entity Too Large" uploading plugins, increase client_max_body_size
       nginx.ingress.kubernetes.io/proxy-body-size: 50m
       nginx.ingress.kubernetes.io/proxy-request-buffering: "off"
   spec:
     tls:
     - hosts:
       - kmadel.k8s.cb-sa.io
       secretName: cb-core-tls
     rules:
     - host: "kmadel.k8s.cb-sa.io"
       http:
         paths:
         - path: /cjoc
           backend:
             serviceName: cjoc
             servicePort: 80
   ```
8. Next use `kubectl` to apply the Kustomize patch changes for the `Ingress`:
   ```
   kubectl apply -k ./kustomize
   ```
9.  Verify that the certificate was created:
    ```
    kubectl -n cb-core get certificate
    ```
10. Now that we successfully created a TLS certificate with the Let's Encrypt staging service we will create a Let's Encrypt production `Issuer`. Make a copy of the ***letsencrypt-staging-issuer.yml*** in the ***cert-manager*** directory name ***letsencrypt-production-issuer.yml*** in the ***cert-manager*** directory and update the `metadata` `name`, `spec` `acme` `server` and `spec` `acme` `privateKeySecretRef` `name` so it matches the following:
    ```yaml
    apiVersion: cert-manager.io/v1alpha2
    kind: Issuer
    metadata:
      name: letsencrypt-production
      namespace: cb-core
    spec:
      acme:
        # You must replace this email address with your own.
        # Let's Encrypt will use this to contact you about expiring
        # certificates, and issues related to your account.
        email: kmadel@cloudbees.com
        server: https://acme-v02.api.letsencrypt.org/directory
        privateKeySecretRef:
          # Secret resource used to store the account's private key.
          name: letsencrypt-production-issuer-account-key
        # Add a single challenge solver, HTTP01 using nginx
        solvers:
        - http01:
            ingress:
              class: nginx
    ```
11. Apply the changes with `kubectl`:
    ```
    kubectl apply -f ./cert-manager/letsencrypt-production-issuer.yml
    ```
12. Next update the cb-core Ingress to use the production Issuer: `cert-manager.io/issuer: "letsencrypt-production"`
13. Apply the changes with `kubectl`:
    ```
    kubectl apply -k ./kustomize
    ```
14. Next