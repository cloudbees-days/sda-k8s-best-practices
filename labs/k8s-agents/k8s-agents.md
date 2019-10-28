# Elastic and Ephemeral Agents with Kubernetes

The Jenkins Kubernetes plugin provides a Kubernetes based agent cloud - and while it is not a proprietary feature of Core there are several Core features that provide enhanced configuration management and monitoring for Kubernetes Cloud agents.

## Default JNLP Base Pod Template

Don't use the OSS Jenkins JNLP container image because:

- It has a JDK installed when all you need is a JRE - this reduces the size significantly and since every single Pod based agent has to have a JNLP container, this will have a big impact on resource utilization and performance. 
- It has a Jenkins slave.jar already installed and this may get out of sync with the Jenkins version on the Managed Master. CloudBees provides a script to pull the slave.jar from the Managed Master that is provisioned from.



## Kubernetes Agent Cloud Specific Namespaces

Security - IAM permissions