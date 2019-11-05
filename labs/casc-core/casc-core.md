# Configuration as Code for CloudBees Core

Configuration as code (CasC) provides audit-ability, governance and cost effective disaster recovery for Jenkins system and job configuration.

Setting up JCasC for Operations Center and Managed Masters required a number of manual steps in the UI. But we want to manage as many configuration aspects of Core as code as possible.

Custom container images work nicely with CasC.

Install plugins.

Install JCasC YAML.

## CasC for Operations Center

Master provisioning.

Shared credentials.

Groovy for other stuff - like the Kubernetes Shared Cloud configuration.

## CasC for Managed Masters

Pipeline shared libraries.

Job configuration.

## JCasC at Scale

Use `yq` to merge a based JCasC yaml file with a Managed Master specific JCasC yaml.

## Automating CasC

Ops Managed Master.

Reprovision Managed Masters when there is a new container image.

Update JCasC when there are updates.