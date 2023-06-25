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

>az storage container create --account-name "$storageName" --name "$secureContainer" --auth-mode login

# Assigning Read and write roles to Azure Function App

>FunctionIdentity=$(az resource list --name $functionAppName --query [*].identity.principalId --out tsv)

>az role assignment create --role "Reader and Data Access" --assignee $FunctionIdentity --scope "/subscriptions/$subscriptionId/resourceGroups/$resourceGroupName/providers/Microsoft.Storage/storageAccounts/$secureStore"

>az role assignment create --role "Storage Blob Data Contributor" --assignee $FunctionIdentity --scope "/subscriptions/$subscriptionId/resourceGroups/$resourceGroupName/providers/Microsoft.Storage/storageAccounts/$secureStore/blobServices/default/containers/$secureContainer"

# storing Azure storage accounts connection strings as a variables in Azure Function

> storageNameConnection=$(az storage account show-connection-string -g $resourceGroupName  -n $storageName --query connectionString --out tsv)

>  secureStoreConnection=$(az storage account show-connection-string -g $resourceGroupName  -n $secureStore --query connectionString --out tsv)

> az functionapp config appsettings set --name "$functionAppName" --resource-group "$resourceGroupName" --settings "storageNameConnectionString=$storageNameConnection"

> az functionapp config appsettings set --name "$functionAppName" --resource-group "$resourceGroupName" --settings "secureStoreConnectionString=$secureStoreConnection"

# Then we have to create a Blob Function in azure function APP
> Choose "Functions" from left blade and click on "Create"
![image](https://github.com/sgrthati/Az-Functions-to-copy-files-from-storage/assets/101870480/cdcf547a-ee31-4092-9343-61112f150a9e)

# Creating a azure blob trigger

> here we have to choose "Azure Blob Storage trigger" then we have to do chnages as per below screenshot

> in below Screenshot,Storage account connection will take a look on given path "container/{blob}.txt",here i intentionally mentioned to look only txt files

![image](https://github.com/sgrthati/Az-Functions-to-copy-files-from-storage/assets/101870480/2b4249ea-9dd1-402d-b6dd-c88b7c820acb)

# here we are implementing whenever new file identified above mentioned storage and path,it will do copy to below mentioned storage

> do changes like below Screenshot

![image](https://github.com/sgrthati/Az-Functions-to-copy-files-from-storage/assets/101870480/415d567a-6565-4563-85db-6c24ca1de6c0)

>we have to enable Az module to work internally we have to do below changes

![image](https://github.com/sgrthati/Az-Functions-to-copy-files-from-storage/assets/101870480/c3e364ff-90e2-40a6-a29d-96820a73c76a)

>It's done,will look on result part
