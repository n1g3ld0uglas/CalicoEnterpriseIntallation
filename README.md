# Calico Enterprise Intallation
Step-by-Step process for installing Calico Enterprise in Air-Gapped AKS Clusters.

## Prerequisite Checks

Enable network mode using the aks-preview extension:
```
az extension add --name aks-preview
az feature register -n AKSNetworkModePreview --namespace Microsoft.ContainerService
az provider register -n Microsoft.ContainerService
```
Create cluster using ```az deployment group create``` with the values:
```
"networkProfile": {
 "networkPlugin": "azure",
 "networkMode": "transparent"
}
```
## Storage

In the following example for an AKS cloud provider integration, the StorageClass tells Calico Enterprise to use LRS disks for log storage.
