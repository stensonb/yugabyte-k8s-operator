
# Yugabyte Kubernetes Operator Documentation

## Overview

The Yugabyte Kubernetes Operator automates the deployment and management of YugabyteDB clusters on
Kubernetes. It goes beyond yugabyteDB  current management automation 
(which relies on REST APIs and GUIs and helm charts).
Establishing a YBDB cluster as a custom resource natively in K8s
Providing k8s methods and native CLIs against the YBDB Cluster for the purpose of backup, upgrade,
and scale out, scale in, changing of gFlags, changing of cpu & memory, and other reconfiguration
activities. 

As part of this first release, we also relase some additonal CRDs to manage the day2 operations of
the yugabytedb universe. 
These include:
  Release CRD - To run multiple releases of yugabytedb and upgrade the software in a universe.
  Support Bundle CRD - To collect logs when universe fails.
  Backup and Restore CRDs - To take full backup of universe and restore.

This documentation is intended for Kubernetes clusters version 1.27 and above and includes details 
on setting up necessary roles and permissions for the service account.

## Prerequisites

- **Kubernetes Cluster**: Version 1.27 or later.
- **Helm**: Version 3 or later.
- **Administrative Access**: Required for the Kubernetes cluster, abilty to create cluster roles, roles, namespaces

## Installation

### Adding the Helm Chart Repository

To add the Yugabyte Helm chart repository:

```shell
helm repo add yugabyte https://charts.yugabyte.com
helm repo update
```

### Installing the Operator with RBAC Permissions

To install the Yugabyte Operator with necessary RBAC permissions:

```shell
kubectl create ns <operator_namespace>
kubectl apply -f crd/concatinated_crds.yaml
helm install -n <operator_namespace> yugabyte-k8s-operator yugabyte/yugabyte-operator --set rbac.create=true
```

This command sets up the necessary Role-Based Access Control (RBAC) permissions, 
including the creation of cluster roles and roles within the service account.

### Verifying the Installation

To verify the installation of the operator:

```shell
kubectl get pods -n <operator_namespace>
[centos@dev-server-anijhawan-4 yugabyte_k8s_operator_chart]$ kubectl get pods -n <operator_namespace> 
NAME                                       READY   STATUS    RESTARTS   AGE
chart-1706728534-yugabyte-k8s-operator-0   3/3     Running   0          26h

```
Check for available releases for yugabyte operator to use. 

```
[centos@dev-server-anijhawan-4 yugabyte-k8s-operator]$ kubectl get releases -n testoperator
NAME                VERSION         STATUS      DOWNLOADED
2.18.6.0-b73        2.18.6.0-b73    Available   true
2.19.3.0-b140       2.19.3.0-b140   Available   true
2.20.1.3-b3         2.20.1.3-b3     Available   true
```






### Service Account

The Yugabyte Operator requires a service account with sufficient permissions to manage resources within the Kubernetes cluster. When installing the operator, ensure that the service account has the necessary roles and cluster roles bound to it.

### Cluster Roles and Roles

- **ClusterRole**: Grants permissions at the cluster-level, necessary for operations that span multiple namespaces or have cluster-wide implications.
- **Role**: Grants permissions within a specific namespace, used for namespace-specific operations.

The operator chart, when installed with `rbac.create=true`, will automatically create appropriate ClusterRoles and Roles.

## Custom Resource Definitions (CRDs)

The operator supports various CRDs for managing YugabyteDB clusters, software releases, backups, and restores. 
Detailed configurations for each CRD are available in the operator's schema, these can be viewed
using kubectl explain or a tool like kubectl explore.

Example
```
[centos@dev-server-anijhawan-4 resources]$ kubectl explain ybuniverse.spec
GROUP:   operator.yugabyte.io
KIND:    YBUniverse
VERSION:  v1alpha1

FIELD: spec <Object>

DESCRIPTION:
  Schema spec for a yugabytedb universe.

FIELDS:
 deviceInfo  <Object>
  Device information for the universe to refer to storage information for
  volume, storage classes etc.

 enableClientToNodeEncrypt   <boolean>
  Enable client to node encryption in the universe. Enable this to use tls
  enabled connnection between client and database.

 enableIPV6  <boolean>
  Enable IPV6 in the universe.

 enableLoadBalancer  <boolean>
  Enable LoadBalancer access to the universe. Creates a service with
  Type:LoadBalancer in the universe for tserver and masters.

 enableNodeToNodeEncrypt    <boolean>
  Enable node to node encryption in the universe. This encrypts the data in
  transit between nodes.

 enableYCQL  <boolean>
  Enable YCQL interface in the universe.

 enableYCQLAuth    <boolean>
  enableYCQLAuth enables authentication for YCQL inteface.

 enableYSQL  <boolean>
  Enable YSQL interface in the universe.

 enableYSQLAuth    <boolean>
  enableYSQLAuth enables authentication for YSQL inteface.

 gFlags    <Object>
  Configuration flags for the universe. These can be set on masters or
  tservers

 kubernetesOverrides  <Object>
  Kubernetes overrides for the universe. Please refer to yugabyteDB
  documentation for more details.
  https://docs.yugabyte.com/preview/yugabyte-platform/create-deployments/create-universe-multi-zone-kubernetes/#configure-helm-overrides

 numNodes   <integer>
  Number of tservers in the universe to create.

 providerName <string>
  Preexisting Provider name to use in the universe.

 replicationFactor   <integer>
  Number of times to replicate data in a universe.

 universeName <string>
  Name of the universe object to create

 ybSoftwareVersion   <string>
  Version of DB software to use in the universe.

 ycqlPassword <Object>
  Used to refer to secrets if enableYCQLAuth is set.

 ysqlPassword <Object>
  Used to refer to secrets if enableYSQLAuth is set.

 zoneFilter  <[]string>
  Only deploy yugabytedb nodes in these zones mentioned in the list. Defaults
  to all zones if unspecified.


[centos@dev-server-anijhawan-4 resources]$ kubectl explain ybuniverse.spec.gFlags
GROUP:   operator.yugabyte.io
KIND:    YBUniverse
VERSION:  v1alpha1

FIELD: gFlags <Object>

DESCRIPTION:
  Configuration flags for the universe. These can be set on masters or
  tservers

FIELDS:
 masterGFlags <map[string]string>
  Configuration flags for the master process in the universe.

 perAZ <map[string]Object>
  Configuration flags per AZ per process in the universe.

 tserverGFlags <map[string]string>
  Configuration flags for the tserver process in the universe.

```

## Getting Started


## Example CRs
To create a universe:
```
apiVersion: operator.yugabyte.io/v1alpha1                                                               
kind: YBUniverse                                                                                        
metadata:                                                                                               
  name: operator-universe-demo                                                                         
spec:                                                                                                                                                                               
  numNodes:    3                                                                                        
  replicationFactor:  1                                                                                 
  enableYSQL: true                                                                                      
  enableNodeToNodeEncrypt: true                                                                         
  enableClientToNodeEncrypt: true                                                                       
  enableLoadBalancer: false 
  ybSoftwareVersion: "2.20.1.3-b3" 
  enableYSQLAuth: false                                                                                 
  enableYCQL: true                                                                                      
  enableYCQLAuth: false                                                                                 
  gFlags:                                                                                               
    tserverGFlags: {}                                                                                   
    masterGFlags: {}                                                                                    
  deviceInfo:                                                                                           
    volumeSize: 400                                                                                     
    numVolumes: 1                                                                                       
    storageClass: "yb-standard"                                                                         
  kubernetesOverrides:                                                                                  
    resource:                                                                                           
      master:                                                                                           
        requests:                                                                                       
          cpu: 2                                                                                        
          memory: 8Gi                                                                                   
        limits:                                                                                         
          cpu: 3                                                                                        
          memory: 8Gi 
```

To add a new software release to use of yugabyteDB
```
apiVersion: operator.yugabyte.io/v1alpha1
kind: Release
metadata:
  name: example-release7
spec:
  config:
    version: "2.19.3.0-b140"
    downloadConfig:
      http:
        paths:
          helmChart: "https://charts.yugabyte.com/yugabyte-2.19.3.tgz" 
```

To create a backup

We first need to create a storage config that describes where the backup will be stored. 
```
apiVersion: operator.yugabyte.io/v1alpha1
kind: StorageConfig
metadata:
  name: sco
spec:
  config_type: STORAGE_S3
  data:
    AWS_ACCESS_KEY_ID: __AWS__ACCESS_KEY 
    AWS_SECRET_ACCESS_KEY: __AWS_SECRET_KEY
    BACKUP_LOCATION:  s3://example-backup-location
```

To create a backup that refers to the above storage config we use the backup CR
```
apiVersion: operator.yugabyte.io/v1alpha1
kind: Backup
metadata:
  name: my-backup
spec:
  backupType: PGSQL_TABLE_TYPE 
  sse: true
  storageConfig: "sco"
  universe: "operator-universe919"
  tableByTableBackup: false
  keyspaceTableList: []
  timeBeforeDelete: 0
```

To Restore the backup from the above backup to an existing universe we use

```
apiVersion: operator.yugabyte.io/v1alpha1
kind: RestoreJob
metadata:
  name: example-restore-job
spec:
  actionType: RESTORE
  universe: example-universe
  storageConfig: example-storage-config
  backupType: YQL_TABLE_TYPE
  keyspaceTableList:
    - keyspace1.table1
    - keyspace2.table2
  backupStorageLocation: example-backup-location
```
## Testing
We have tested this operator on GKE, EKS, AKS, it needs the following labels defined on the nodes to
work correctly. 
          ```
          failure-domain.beta.kubernetes.io/region: us-west1
          failure-domain.beta.kubernetes.io/zone: us-west1-b
          ```

## Additional Information

- Ensure Kubernetes cluster version is 1.27 or higher.
- CRDs are located in a file named `./crd/concatenated_crd.yaml`
- The operator is provided as a Helm chart.

## Support

For assistance or issues related to the Yugabyte Kubernetes Operator, please reach out to us on slack 
https://communityinviter.com/apps/yugabyte-db/register
```
