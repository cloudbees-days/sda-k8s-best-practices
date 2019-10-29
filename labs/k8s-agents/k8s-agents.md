# Elastic and Ephemeral Agents with Kubernetes

The Jenkins Kubernetes plugin provides a Kubernetes based agent cloud - and while it is not a proprietary feature of Core there are several Core features that provide enhanced configuration management and monitoring for Kubernetes Cloud agents.

## Default JNLP Base Pod Template

Don't use the OSS Jenkins JNLP container image because:

- It has a JDK installed when all you need is a JRE - this reduces the size significantly and since every single Pod based agent has to have a JNLP container, this will have a big impact on resource utilization and performance. 
- It has a Jenkins slave.jar already installed and this may get out of sync with the Jenkins version required by the Managed Master provisioning the agent. CloudBees installs a script as a Kubernetes ConfigMap that can be configured to be mounted to all agent Pods and will pull the slave.jar from the Managed Master that it is being provisioned from before creating an agent connection to the Managed Master.

### Update the Kubernetes Shared Cloud

As part of the CloudBees Core install a Pod Template is created for you.

## Kubernetes Agent Cloud Specific Namespaces

Security - IAM permissions