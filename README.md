# Bookstack Backup using Azure Pipelines

Azure DevOps pipelines to make a backup of Bookstack application deployed on a Kubernetes cluster using [this Helm chart](https://github.com/pacroy/bookstack-helm) and upload backup archives to Azure storage account.

Supported Authentications:

- Azure Kubernetes Service (AKS) with [Azure Active Directory integration](https://docs.microsoft.com/en-us/azure/aks/managed-aad)
- Kubernetes cluster with generic user token authentication

## Service Principal

An Azure AD service principal is required to access AKS cluster and Azure storage account.

### Create Service Principal

```sh
az ad sp create-for-rbac --skip-assignment --name "http://your-sp-name"
```

### Grant Permission to Storage Account

Contributor permission is required over a storage account to upload backup archives.

```sh
az role assignment create --assignee "{service-principal-id}" --role "Contributor" --scope "/subscriptions/{subscription-id}/resourceGroups/{resource-group}/providers/Microsoft.Storage/storageAccounts/{storage-account}"
```

### Grant Permission to AKS Cluster

`Azure Kubernetes Service Cluster User Role` permission is required to get the cluster credential (kubeconfig)

```sh
az role assignment create --assignee "{service-principal-id}" --role "Azure Kubernetes Service Cluster User Role" --scope "/subscriptions/{subscription-id}/resourcegroups/{resource-group}/providers/Microsoft.ContainerService/managedClusters/{aks-cluster}"
```

List Permissions to verify

```sh
az role assignment list --all --assignee "{service-principal-id}" --subscription "{subscription-id}" --output table
```

### Grant Permission to AKS Namespace

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

| Name | Description | AKS | Generic |
|---|---|---|---|
| TENANT_ID | Azure Tenant ID of service principal | X | X |
| SERVICE_PRINCIPAL_ID | Service principal client ID that having permission to access AKS and storage account | X | X |
| SERVICE_PRINCIPAL_SECRET | Service principal client secret | X | X |
| KUBE_CA_BASE64 | Certificate authority data from kubeconfig file | X | X |
| KUBE_API_SERVER | Kubernetes API server URL:port | X | X |
| KUBE_SERVER_ID | AKS server Azure AD application ID | X | |
| KUBE_CLIENT_ID | AKS client Azure AD application ID | X | |
| KUBE_USER_TOKEN | User token from kubeconfig file | | X |
| KUBE_CONTEXT | Kubernetes context/cluster Name | X | X |
| WIKI_NAMSPACE | BookStack Namespace | X | X |
| MYSQL_APP_LABEL | MySQL application label i.e. `<release_name>-mysql` | X | X |
| BOOKSTACK_APP_LABEL | Bookstack appplication label i.e. `<release_name>-bookstack` | X | X |
| STORAGE_SUBSCRIPTION_ID | _[Optional]_ Subscription ID of the storage account | X | X |
| STORAGE_ACCOUNT_NAME | Azure Storage Account Name | X | X |
| BLOB_CONTAINER_NAME | Azure Storage Blob Container Name | X | X |
