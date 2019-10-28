# Managed Master Provisioning and Configuration

The dynamic provisioning of Managed Master by Operations Center is one of the most important features for CloudBees Core on Kubernetes. The ability to quickly and easily provision two pizza team Managed Masters - that are also easier to manage than standalone OSS Jenkins Masters - provides enhance stability, security and better performance per team.

## Master Specific Namespaces

Primarily for security, but also allows for tracking cloud costs at the team level.

## JCasC at Scale

Use `yq` to merge a based JCasC yaml file with a Managed Master specific JCasC yaml.