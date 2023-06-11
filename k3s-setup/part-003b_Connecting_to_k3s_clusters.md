
Connecting to the k3s nodes is a bit more challenging.


#### Prequisites

| Item | requirement| additional notes |
|root access| must have root access to complete the configuration| when setting up Ubuntu we created the login "k3s" and a password. |

#### Step 1 ( on Management Server)
On your Management server 
[Install Kubectl and kubelogin](/k3s-setup/Part-003-PreRequisites.md)

#### Step 2 
##### This step involves modifying the k3s kubeconfig credentials to access the cluster remotely.
Here is a summary of the action.
a. On the k3s node create a copy of the `/etc/rancher/k3s/k3s.yaml` to `~/.kube/config`
b. Update  `~/.kube/config` to reflect correct clusternames and IP address.
c. Copy this file to the management server and configure $KUBECONFIG file from the remote cluster.

##### step a ( on k3s node)

Create copy of `/etc/rancher/k3s/k3s.yaml`
```
mkdir ~/.kube/ -p  &&  sudo cat /etc/rancher/k3s/k3s.yaml > ~/.kube/config
```
Validate if files is copied correctly.
```
cat ~/.kube/config
```
##### step b ( on k3s node)
Replace IP address of server from 127.0.0.1 to IP or hostname of local server
```
## to replace 127.0.0.1 to IP 192.168.0.200    
sed -i 's/127\.0\.0\.1/192\.168\.0\.200/g' ~/.kube/config  
```
#### To replace the other default names, you will need yq installed.
Declare the kubeconfig file location
```
file_path=~/.kube/config
```
#### Update the cluster name field in the users section

#### Change Cluster Name
```
yq eval '.clusters[0].name = "k3s-single-01-cluster"' -i "$file_path"
```
#### Change the user name field in the users section
#### to set cluster user
```
yq eval '.users[0].name = "k3s-single-01-user"' -i "$file_path"
```
#### The next to create the context associating the user associated with the cluster

#### Change the context name field
#### set context Name
```
yq eval '.contexts[0].name = "k3s-single-01-cluster-context"' -i "$file_path"
```

#### Set contexts with Cluster and user names with values used above. 
```
yq eval '.contexts[0].context.cluster = "k3s-single-01-cluster"' -i "$file_path"
yq eval '.contexts[0].context.user = "k3s-single-01-user"' -i "$file_path"
```

```
## here is a bash script 
#!/bin/bash
#Update your variables here.

export CLUSTER="k3s-single-01-cluster"
export USER="k3s-single-01-user"
export CONTEXT=k3s-single-01-cluster-context

file_path=~/.kube/config
#### Update the cluster name field in the users section
#### Change Cluster Name
yq eval '.clusters[0].name = strenv(CLUSTER)' -i "$file_path"
#### Change the user name field in the users section
#### to set cluster user
yq eval '.users[0].name = strenv(USER)' -i "$file_path"
#### The next to create the context associating the user associated with the cluster

#### Change the context name field
#### set context Name
yq eval '.contexts[0].name = strenv(CONTEXT)' -i "$file_path"

#### Set contexts with Cluster and user names with values used above. 
yq eval '.contexts[0].context.cluster = strenv(CLUSTER)' -i "$file_path"
yq eval '.contexts[0].context.user = strenv(USER)' -i "$file_path"

```

####  You can now check the output ( use the -v=6 to check the location of the kubectl file used)

```
root@k3s-single-01:~# kubectl config get-contexts -v=6
I0611 09:06:29.080862    8662 loader.go:373] Config loaded from file:  /root/.kube/config
CURRENT   NAME                            CLUSTER                 AUTHINFO             NAMESPACE
*         k3s-single-01-cluster-context   k3s-single-01-cluster   k3s-single-01-user
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
