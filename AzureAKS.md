# Azure Kubernetes Service

Setup time ~ 7 minutes

## Install Azure CLI

```
brew update && brew install azure-cli
```
Ref: Install Azure CLI https://docs.microsoft.com/en-us/cli/azure/install-azure-cli 

## login 
```
az login
```

## create cluster
```
az aks create -g anton-rg --location eastus \
  --name anton-aks-rec --node-count 3 \
  --node-vm-size Standard_D4_v3
```
Ref: az aks create documentation https://docs.microsoft.com/en-us/cli/azure/aks?view=azure-cli-latest#az-aks-create recommended node size - at least 4 cores/16GB RAM

Ref: VM types/sizes: https://docs.microsoft.com/en-us/azure/virtual-machines/sizes

## add cluster credentials to kubectl
```
az aks get-credentials -g anton-rg  --name anton-aks-rec
```
## delete cluster
```
az aks delete -g anton-rg --name anton-aks-rec
```
Note: this takes surprisingly long time!
## Clear up kubectl context
```
kubectl config delete-context anton-aks-rec
```