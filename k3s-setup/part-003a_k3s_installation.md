
## prerequisite.
#####################prerequisite#############################
########## Install kubelogin and kubectl.
#### easiest method and enable future updates easier without other tools like brew. 
https://learn.microsoft.com/en-us/cli/azure/install-azure-cli-linux?pivots=apt

Install AZ CLI per Prerequisites
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



#### setup k3s no ad login ####
Setup basic disable traefic
### config stored in 
```
curl -sfL https://get.k3s.io | \
  sh -s - \
  --write-kubeconfig-mode 644 \
  --disable "traefik"  
```


Setup with AAD login 
### config stored in 
```
curl -sfL https://get.k3s.io | \
  sh -s - \
  --write-kubeconfig-mode 644 \
  --disable "traefik"  \
  --kube-apiserver-arg "authorization-mode=RBAC" \
  --kube-apiserver-arg "oidc-client-id=spn:xxxxxxxxxxx" # kubeapi app ID
  --kube-apiserver-arg "oidc-issuer-url=https://sts.windows.net/$TENANTID/" \
  --kube-apiserver-arg "oidc-username-claim=upn" \
  --kube-apiserver-arg "oidc-groups-claim=groups" \
  --kube-apiserver-arg "oidc-username-prefix=aad:" \
  --kube-apiserver-arg "oidc-groups-prefix=aad:"
```  
