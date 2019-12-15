# Azure Image Builder
----------------------------------------------------------------------------

Get the list of Ubuntu images you can use.

```bash
az vm image list -l eastus -f UbuntuServer -p Canonical --output table –-all
```

Setup the environment.

```bash
{
  imageResourceGroup=azbuilder-eus-rg
  location=EastUS
  subscriptionID=$(az account show --query "id" --output tsv)
  imageName=k8sBase
  runOutputName=k8s
}
```

Now create your resource group and assign the Azure Image builder service permissions to that resource group.

```bash
az group create -n $imageResourceGroup -l $location -o none
```


```bash
az role assignment create \
    --assignee cf32a0cc-373c-47c9-9156-0db11f6a6dfc \
    --role Contributor \
    --scope /subscriptions/$subscriptionID/resourceGroups/$imageResourceGroup \
    --output none
```

Now we need to substitute our variables into our template.

```bash
{    
  sed -i -e "s/<subscriptionID>/$subscriptionID/g" baseKubernetes.json
  sed -i -e "s/<rgName>/$imageResourceGroup/g" baseKubernetes.json
  sed -i -e "s/<region>/$location/g" baseKubernetes.json
  sed -i -e "s/<imageName>/$imageName/g" baseKubernetes.json
  sed -i -e "s/<runOutputName>/$runOutputName/g" baseKubernetes.json
}
```

Upload the template to your resource group

```bash
az resource create \
    --resource-group $imageResourceGroup \
    --properties @baseKubernetes.json \
    --is-full-object \
    --resource-type Microsoft.VirtualMachineImages/imageTemplates \
    -n baseKubernetes
```

Now all we need to do is run it.

```bash
az resource invoke-action \
     --resource-group $imageResourceGroup \
     --resource-type  Microsoft.VirtualMachineImages/imageTemplates \
     -n baseKubernetes \
     --action Run 
```    

Once this completes running we should have an image being displayed in the *azbuilder-eus-rg* that we created earlier.

Let's create a VM from that image and see if the correct versions of our software are installed and configured to be locked.

```bash
az vm create \
  --resource-group $imageResourceGroup \
  --name k8sNode1 \
  --admin-username aibuser \
  --image $imageName \
  --location $location \
  --generate-ssh-keys
```

Now let's SSH into that VM.

```bash
ssh aibuser@$(az vm show -g $imageResourceGroup -n k8sNode1 -d --query "publicIps" --output tsv)
```

Now we can execute a few tests.

```bash
$ kubectl version
Client Version: version.Info{Major:"1", Minor:"16", GitVersion:"v1.16.4", GitCommit:"224be7bdce5a9dd0c2fd0d46b83865648e2fe0ba", GitTreeState:"clean", BuildDate:"2019-12-11T12:47:40Z", GoVersion:"go1.12.12", Compiler:"gc", Platform:"linux/amd64"}
```

```bash
$ kubeadm version
kubeadm version: &version.Info{Major:"1", Minor:"16", GitVersion:"v1.16.4", GitCommit:"224be7bdce5a9dd0c2fd0d46b83865648e2fe0ba", GitTreeState:"clean", BuildDate:"2019-12-11T12:44:45Z", GoVersion:"go1.12.12", Compiler:"gc", Platform:"linux/amd64"}
```

```bash
$ kubelet --version
Kubernetes v1.16.4
```

```bash
$ docker version
Client:
 Version:           18.06.1-ce
 API version:       1.38
 Go version:        go1.10.3
 Git commit:        e68fc7a
 Built:             Tue Aug 21 17:24:51 2018
 OS/Arch:           linux/amd64
 Experimental:      false
```

```bash
$ sudo apt-mark showhold
docker-ce
kubeadm
kubectl
kubelet
```

All those tests look great. We could now use this image to create Kubernetes v1.16.4 clusters.

Here is how we can cleanup the template if needed.

```bash
 az resource delete \
     --resource-group $imageResourceGroup \
     --resource-type Microsoft.VirtualMachineImages/imageTemplates \
     -n baseKubernetes
```     



Latest tag will not work.     
     
 Looks like you don't need to deprovision when  using the image builder as it does it automagically.  https://docs.microsoft.com/en-us/azure/virtual-machines/linux/image-builder-json?toc=%2Fazure%2Fvirtual-machines%2Fwindows%2Ftoc.json#generalize**  