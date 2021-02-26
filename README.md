# Bookstack Backup using ADO

Azure DevOps pipeline to backup Bookstack wiki deployed on AKS using [this Helm chart](https://github.com/pacroy/bookstack).

## Variables

| Name | Description |
|---|---|
| TENANT_ID | Azure Tenant ID of service principal |
| SERVICE_PRINCIPAL_ID | Service principal client ID that having `cluster-admin` over the AKS and `Contributor` over the storage account |
| SERVICE_PRINCIPAL_SECRET | Service principal client secret |
| AKS_RESOURCE_GROUP | Resource Group Name of AKS |
| AKS_CLUSTER_NAME | AKS Cluster Name |
| KUBE_CONTEXT | Kubectl Context Name |
| WIKI_NAMSPACE | Kubectl Namespace |
| MYSQL_APP_LABEL | MySQL app label i.e. `<release_name>-mysql` |
| BOOKSTACK_APP_LABEL | Bookstack app label i.e. `<release_name>-bookstack` |
| STORAGE_ACCOUNT_NAME | Azure Storage Account Name |
| BLOB_CONTAINER_NAME | Azure Storage Blob Container Name |
