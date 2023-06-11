## Prequisites
#### Install kubelogin and kubectl on the management server
#### There are several ways to install kubectl and kubelogin. I find the following the easiest as they can be updated via the az aks updates.

[Install AZ CLI per Prerequisites](https://learn.microsoft.com/en-us/cli/azure/install-azure-cli-linux?pivots=apt)

```
cd /usr/bin/
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
```
If you get errors try this 
```
sudo vi /etc/apt/sources.list
# remark  deb file:/cdrom focal Release
```

Execute the commands below to install kubectl and kubelogin

```
sudo az aks install-cli  # installs kubectl and kubelogin.
kubectl version --client
kubelogin --version
```


#####################prerequisite-end#############################
