
Connecting to the k3s nodes is a bit more challenging.


#### Prequisites

| Item | requirement| additional notes |
|root access| must have root access to complete the configuration| when setting up Ubuntu we created the login "k3s" and a password. |

#### Step 1 
########## Install kubelogin and kubectl on the management server
#### easiest method and enable future updates easier without other tools like brew. 
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

#### Step 2
##### This step involves copying the k3s kubeconfig credentials to access the cluster remotely.

The key is to have a login that has access to the file .
```
/etc/rancher/k3s/k3s.yaml
```

######  Run commands on the k3s cluster locally.


copy remote file `/etc/rancher/k3s/k3s.yaml` 


Commands run on the management server 
copy file on remote server to a read permission location i.e. remote profile of user copying file
```
cp /etc/rancher/k3s/k3s.yaml ~/.kube/config
 
# creates folder ~/.kube/remote and copy k3s.yaml to created folder
mkdir ~/.kube/ -p  &&  sudo cat /etc/rancher/k3s/k3s.yaml > ~/.kube/config

### replace IP address of server from 127.0.0.1 to IP or hostname of local server
remotek3sIP="192.168.0.206"
remotek3sIPpattern="192\.168\.0\.206"
id="k3s"

## to replace 127.0.0.1 to IP 192.168.0.20    
sed -i 's/127\.0\.0\.1/192\.168\.0\.206/g' ~/.kube/config  

```

#update default to relevant names
``` bash
## did not help KUBECONFIG='~/.kube/config'
kubectl config view  --kubeconfig ~/.kube/config
kubectl config rename-context "default" "k3s-single-01-cluster" --kubeconfig ~/.kube/config

#rename defaults to appropriate values 
# yq install using brew
#!/bin/bash

file_path=~/.kube/config

# Update the cluster name field in the users section
# to set cluster name
yq eval '.clusters[0].name = "k3s-single-01-cluster"' -i "$file_path"

# Update the user name field in the users section
# to set cluster user
yq eval '.users[0].name = "k3s-single-01-user"' -i "$file_path"
# thje next to create the context associating the user associated with the cluster
# Update the context name field
# to set contest user
yq eval '.contexts[0].name = "k3s-single-01-cluster-context"' -i "$file_path"
# Update the user name field
# to set contexts with Cluster 
yq eval '.contexts[0].context.cluster = "k3s-single-01-cluster"' -i "$file_path"
yq eval '.contexts[0].context.user = "k3s-single-01-user"' -i "$file_path"

```

```
#output after the changes
k3s@k3s-single-01:~$ kubectl config get-contexts --kubeconfig ~/.kube/config
CURRENT   NAME                            CLUSTER                 AUTHINFO             NAMESPACE
k3s-single-01-cluster-context   k3s-single-01-cluster   k3s-single-01-user
aaa

```


```
kubectl config view --kubeconfig ~/.kube/config --raw --minify --flatten --output jsonpath='{.users[?(@.name=="default")].user.client-certificate-data}' > cert_data.txt

kubectl config set-credentials k3s-single-01-user --client-certificate=cert_data.txt --kubeconfig ~/.kube/config


```

### Step 3

Run on remote
Use scp to copy file from remote server.
```
Copy edited kubectl from remote server
scp k3s@192.168.0.206:~/.kube/config ~/.kube/remote/config-206

```



replace 127.0.0.1 with IP of remote cluster on the config file copied.

```
extract Cert data to be updated in new kubeconfig
- certificate-authority-data
kubectl config view --kubeconfig ~/.kube/remote/config-206 --raw --minify --flatten --output jsonpath='{.clusters[?(@.name=="default")].user.client-certificate-data}' > /tmp/cert_certificate-authority-data.txt

- client-certificate-data
kubectl config view --kubeconfig ~/.kube/remote/config-206 --raw --minify --flatten --output jsonpath='{.users[?(@.name=="k3s-single-01-user")].user.client-certificate-data}' > /tmp/cert_client-certificate-data.txt

- client-key-data
kubectl config view --kubeconfig ~/.kube/remote/config-206 --raw --minify --flatten --output jsonpath='{.users[?(@.name=="k3s-single-01-user")].user.client-key-data}' > /tmp/cert_client-key-data.txt
 
```


```
update kubeconfig with data from 
Merge multiple kubeconfig files.
Place all config files copied from remote servers to ~/.kube/remote/

run command to merge the 
export KUBECONFIG=~/.kube/:$(find ~/.kube/remote/ -type f | tr '\n' ':')


```
