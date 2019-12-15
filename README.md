azure-image-builder

az vm image list -l eastus -f UbuntuServer -p Canonical --output table –-all

imageResourceGroup=azbuilder-eus-rg
location=EastUS
subscriptionID=$(az account show --query "id" --output tsv)
imageName=k8sBase
runOutputName=k8s

az group create -n $imageResourceGroup -l $location -o none
az role assignment create \
    --assignee cf32a0cc-373c-47c9-9156-0db11f6a6dfc \
    --role Contributor \
    --scope /subscriptions/$subscriptionID/resourceGroups/$imageResourceGroup \
    --output none

{    
  sed -i -e "s/<subscriptionID>/$subscriptionID/g" baseKubernetes.json
  sed -i -e "s/<rgName>/$imageResourceGroup/g" baseKubernetes.json
  sed -i -e "s/<region>/$location/g" baseKubernetes.json
  sed -i -e "s/<imageName>/$imageName/g" baseKubernetes.json
  sed -i -e "s/<runOutputName>/$runOutputName/g" baseKubernetes.json
}

az resource create \
    --resource-group $imageResourceGroup \
    --properties @baseKubernetes.json \
    --is-full-object \
    --resource-type Microsoft.VirtualMachineImages/imageTemplates \
    -n baseKubernetes
    
az resource invoke-action \
     --resource-group $imageResourceGroup \
     --resource-type  Microsoft.VirtualMachineImages/imageTemplates \
     -n baseKubernetes \
     --action Run 
     
 az resource delete \
     --resource-group $imageResourceGroup \
     --resource-type Microsoft.VirtualMachineImages/imageTemplates \
     -n baseKubernetes
     
     
 Looks like you don't need to deprovision when  using the image builder as it does it automagically.  https://docs.microsoft.com/en-us/azure/virtual-machines/linux/image-builder-json?toc=%2Fazure%2Fvirtual-machines%2Fwindows%2Ftoc.json#generalize**  