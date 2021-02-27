# Bookstack Backup using ADO

Azure DevOps pipelines to make a backup of Bookstack application deployed on AKS using [this Helm chart](https://github.com/pacroy/bookstack) and upload backup archives to Azure storage account.

## Service Principal

This pipelines needs a service principal to access AKS cluster and Azure storage account.

### Create Service Principal

```sh
az ad sp create-for-rbac --skip-assignment --name "http://your-sp-name"
```

### Grant Permission to Storage Account

Contributor permission is required over a storage account to upload backup archives.

```sh
az role assignment create --assignee "{service-principal-id}" --role "Contributor" --scope "/subscriptions/{subscription-id}/resourceGroups/{resource-group}/providers/Microsoft.Storage/storageAccounts/{storage-account}"
```

### Grant Permission to AKS cluster

`Azure Kubernetes Service Cluster User Role` permission is required to get the cluster credential (kubeconfig)

```sh
az role assignment create --assignee "{service-principal-id}" --role "Azure Kubernetes Service Cluster User Role" --scope "/subscriptions/{subscription-id}/resourcegroups/{resource-group}/providers/Microsoft.ContainerService/managedClusters/{aks-cluster}"
```

List Permissions to verify

```sh
az role assignment list --all --assignee "{service-principal-id}" --subscription "{subscription-id}" --output table
```

### Grant Permission to Namespace

To access Bookstack in a namespace, you also need to create ClusterRoleBinding with permission `edit`.

Get service principal object id.

```sh
az ad sp show --id "{service-principal-id}" --query "objectId" --output tsv
```

Set service principal object id in [`clusterrolebinding.yml`](clusterrolebinding.yml) and apply it.

```sh
kubectl apply -f clusterrolebinding.yml
```

## Variables

| Name | Description |
|---|---|
| TENANT_ID | Azure Tenant ID of service principal |
| SERVICE_PRINCIPAL_ID | Service principal client ID that having permission to access AKS and storage account |
| SERVICE_PRINCIPAL_SECRET | Service principal client secret |
| AKS_RESOURCE_GROUP | Resource Group Name of AKS |
| AKS_CLUSTER_NAME | AKS Cluster Name |
| KUBE_CONTEXT | Kubectl Context Name |
| WIKI_NAMSPACE | Kubectl Namespace |
| MYSQL_APP_LABEL | MySQL app label i.e. `<release_name>-mysql` |
| BOOKSTACK_APP_LABEL | Bookstack app label i.e. `<release_name>-bookstack` |
| STORAGE_ACCOUNT_NAME | Azure Storage Account Name |
| BLOB_CONTAINER_NAME | Azure Storage Blob Container Name |
