azure-image-builder

imageResourceGroup=azbuilder-eus-rg
location=EastUS
subscriptionID=$(az account show --query "id" --output tsv)
imageName=k8s-base
runOutputName=k8s

az group create -n $imageResourceGroup -l $location -o none
az role assignment create \
    --assignee cf32a0cc-373c-47c9-9156-0db11f6a6dfc \
    --role Contributor \
    --scope /subscriptions/$subscriptionID/resourceGroups/$imageResourceGroup \
    --output none