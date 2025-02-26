# Multi-tenancy

CAPOCI supports multi-tenancy wherein different OCI user principals can be used to reconcile 
different OCI clusters. This is achieved by associating a cluster with a Cluster Identity and
associating the identity with a user principal. Currently only OCI user principal is supported
for Cluster Identity.

# Steps

## Step 1 - Create a secret with user principal in the management cluster

Please read the [doc][iam-user] to know more about the parameters below.

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: user-credentials
  namespace: default
type: Opaque
data:
  tenancy: <base-64-encoded value of tenancy OCID>
  user: <base-64-encoded value of user OCID>
  key: <base-64-encoded value of user Key>
  fingerprint: <base-64-encoded value of fingerprint>
  passphrase: <base-64-encoded value of passphrase. This is optional>
  region: <base-64-encoded value of region. This is optional>
```

## Step 2 - Edit the cluster template to add a Cluster Identity section and point the OCICluster to the Cluster Identity

The Cluster Identity should have a reference to the secret created above.

```yaml
---
kind: OCIClusterIdentity
metadata:
  name: cluster-identity
  namespace: default
spec:
  type: UserPrincipal
  principalSecret:
    name: user-credentials
    namespace: default
  allowedNamespaces: {}
---
apiVersion: infrastructure.cluster.x-k8s.io/v1beta2
kind: OCICluster
metadata:
  labels:
    cluster.x-k8s.io/cluster-name: "${CLUSTER_NAME}"
  name: "${CLUSTER_NAME}"
spec:
  compartmentId: "${OCI_COMPARTMENT_ID}"
  identityRef:
    apiVersion: infrastructure.cluster.x-k8s.io/v1beta2
    kind: OCIClusterIdentity
    name: cluster-identity
    namespace: default
```

# allowedNamespaces

`allowedNamespaces` can be used to control which namespaces the `OCIClusters` are allowed to use the identity from. 
Namespaces can be selected either using an array of namespaces or with label selector.
An empty `allowedNamespaces` object indicates that `OCIClusters` can use this identity from any namespace.
If this object is `nil`, no namespaces will be allowed, which is the default behavior of the field if not specified.
> Note: NamespaceList will take precedence over Selector if both are set.

## Cluster Identity using Instance Principals

Cluster Identity also supports [Instance Principals][instance-principals]. The example `OCIClusterIdentity`
spec shown below uses Instance Principals.

```yaml
---
kind: OCIClusterIdentity
metadata:
  name: cluster-identity
  namespace: default
spec:
  type: InstancePrincipal
  allowedNamespaces: {}
```

[iam-user]: https://docs.oracle.com/en-us/iaas/Content/API/Concepts/apisigningkey.htm#Required_Keys_and_OCIDs
[instance-principals]: https://docs.oracle.com/en-us/iaas/Content/Identity/Tasks/callingservicesfrominstances.htm