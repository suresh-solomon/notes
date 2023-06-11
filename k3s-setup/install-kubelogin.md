Install AZ CLI per Prerequisites

    cd /usr/bin/
    curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash

Execute the commands per doc to install kubectl and kubelogin

```
sudo az aks install-cli  ## install kubectl and kubelogin
sudo kubectl version --client
sudo kubelogin --version
```
