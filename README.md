# Vault Enterprise Redundancy Zones on Kubernetes

A sample set of values for the
[Vault Helm chart](https://github.com/hashicorp/vault-helm/tree/main) to deploy
a Vault Enterprise cluster with six nodes across multiple availability zones and
with
[Enterprise Redundancy Zones](https://developer.hashicorp.com/vault/docs/enterprise/redundancy-zones)
enabled.

This solution implements the following:

1. **Topology Spread Constraints:** Ensure that individual cluster nodes are
   balanced across availability zones.
2. **Init Container:** An initContainer in each pod that queries the AWS
   Instance Metadata Service for the Availability Zone in which the
   corresponding host is running and writes it to a file in a shared volume.
3. **Post-Start Script:** A script that reads the Availability Zone from a
   temporary file in the shared volume, updates the server configuration, and
   notifies the Vault process to reload its configuration (SIGHUP).

The following additional settings are configured in `values.dev.yaml` to make it
easier to deploy this solution in a non-production environment:

1. **Rolling Updates:** Pods are automatically updated when the StatefulSet
   definition changes.
2. **Pod Anti-Affinity:** Changing the chart default's pod anti-affinity from
   "required" to "preferred", so that a cluster node may be scheduled to the
   same host as another cluster node (reducing the minimum number of nodes in
   the EKS cluster).

While this solution should work on any Kubernetes cluster, it was tested on an
AWS EKS cluster with nodes deployed across three AWS Availability Zones in the
same AWS Region.

## Pre-Requisites

- EKS Cluster
- Default Storage Class configured
- Storage controller, e.g. AWS EBS CSI Driver, as applicable
- Helm release is named `vault` (or labels in values files updated accordingly)
