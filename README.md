# Az-Functions-to-copy-files-from-storage
#will go with Az Login Initially
> Az Login

# Setup Variables

>subscriptionId=$(az account show --query id -o tsv)

>resourceGroupName="Function-App"

>storageName="storagefuncsag$RANDOM"

>functionAppName="azurefunctionsag$RANDOM"

>region="eastus"

>secureStore="destsag$RANDOM"

>secureContainer="storeddata"

# Creating Resource Group

> az group create --name "$resourceGroupName" --location "$region"

# Azure storage account for to store files from source storage account

>az storage account create --name "$secureStore" --location "$region" --resource-group "$resourceGroupName" --sku "Standard_LRS" --kind "StorageV2" --https-only true --min-tls-version "TLS1_2"

# Azure storage account linked with Azure function app (azure function moniters this storage for new Blobs)

>az storage account create --name "$storageName" --location "$region" --resource-group "$resourceGroupName" --sku "Standard_LRS" --kind "StorageV2" --https-only true --min-tls-version "TLS1_2"

# Creating Azure Function App

>az functionapp create --name "$functionAppName" --storage-account "$storageName" --consumption-plan-location "$region" --resource-group "$resourceGroupName" --os-type "Windows" --runtime "powershell" --runtime-version "7.2" --functions-version "4" --assign-identity

# Adding Storage Blob Contributer Role to storage and adding container to it

>Userid=$(az ad signed-in-user show --query id -o tsv)

>az role assignment create --role "Storage Blob Data Contributor" --assignee $Userid --scope "/subscriptions/$subscriptionId/resourceGroups/$resourceGroupName/providers/Microsoft.Storage/storageAccounts/$secureStore"

>az storage container create --account-name "$secureStore" --name "$secureContainer" --auth-mode login

# Assigning Read and write roles to Azure Function App

>FunctionIdentity=$(az resource list --name $functionAppName --query [*].identity.principalId --out tsv)

>az role assignment create --role "Reader and Data Access" --assignee $FunctionIdentity --scope "/subscriptions/$subscriptionId/resourceGroups/$resourceGroupName/providers/Microsoft.Storage/storageAccounts/$secureStore"

>az role assignment create --role "Storage Blob Data Contributor" --assignee $FunctionIdentity --scope "/subscriptions/$subscriptionId/resourceGroups/$resourceGroupName/providers/Microsoft.Storage/storageAccounts/$secureStore/blobServices/default/containers/$secureContainer"

# storing Azure storage accounts connection strings as a variables in Azure Function

> storageNameConnection=$(az storage account show-connection-string -g $resourceGroupName  -n $storageName --query connectionString --out tsv)

>  secureStoreConnection=$(az storage account show-connection-string -g $resourceGroupName  -n $secureStore --query connectionString --out tsv)

> az functionapp config appsettings set --name "$functionAppName" --resource-group "$resourceGroupName" --settings "storageNameConnectionString=$storageNameConnection"

> az functionapp config appsettings set --name "$functionAppName" --resource-group "$resourceGroupName" --settings "secureStoreConnectionString=$secureStoreConnection"

