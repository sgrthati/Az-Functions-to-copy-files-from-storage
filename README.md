# Az-Functions-to-copy-files-from-storage
#will go with Az Login Initially
> Az Login

#Setup Variables

>subscriptionId=$(az account show --query id -o tsv)

>$resourceGroupName="Function-App"

>$storageName="storagefuncsag$RANDOM"

>$functionAppName = "azurefunctionsag$RANDOM"

>$region = "eastus"

>$secureContainer = "storeddata"
