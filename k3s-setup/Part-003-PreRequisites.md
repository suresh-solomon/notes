## Prequisites install kubectl and kubelogin
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
## Prequisites install yq
Get the latest url from [official yq github repo](https://github.com/mikefarah/yq/releases)

To install yq [version 4.34.1](https://github.com/mikefarah/yq/releases/tag/v4.34.1)
Run the following command.
```
wget https://github.com/mikefarah/yq/releases/download/v4.34.1/yq_linux_amd64 -O /usr/bin/yq &&\
    chmod +x /usr/bin/yq
```

#####################prerequisite-end#############################
