https://medium.com/@olemarkus/using-azure-ad-to-authenticate-to-kubernetes-eb143d3cce10

1. step one create ad groups.

2. Create new app registration >> this is the API server
az ad app create --display-name k3sapiserver
fill in following from az portal.

--oidc-client-id="spn:<application id>" \
--oidc-issuer-url="https://sts.windows.net/<azure AD tenant>/"
--oidc-username-claim="upn"


--oidc-client-id="spn:ab037680-d81b-4690-b277-a39b45660a28" \
--oidc-issuer-url="https://sts.windows.net/a01c77f7-d822-4d88-88fd-9a738c46e3ab/"
--oidc-username-claim="upn"

also need to expose API for th next app use.

3. Create another new app registration


az ad app create --display-name kubectl
 Go to the API Permissions tab, 
 click Add a permission.
 Pick the My APIs.
 Search for the app created for the API server and 
 add the user impression permission.



4. get CA from Cluster.

Now pick a lucky AD user that will be granted access to your cluster. Provide him the cluster CA certificate. If you provisioned the cluster, Kops would have created a kubectl configuration file for you with a base64 encoded version of the CA certificate. This should be located under:
kubectl config set-cluster <cluster name> --server=https://api.<cluster name> --certificate-authority=/tmp/<cluster name>.crt


updated the kubeconfig 

kubectl config set-credentials <upn>\
--auth-provider=azure \
--auth-provider-arg=environment=AzurePublicCloud \
--auth-provider-arg=client-id=<kubectl application id>\
--auth-provider-arg=tenant-id=<Azure AD tenant>\
--auth-provider-arg=apiserver-id=<api server application id>

kubectl config set-credentials <upn>\
--auth-provider=azure \
--auth-provider-arg=environment=AzurePublicCloud \
--auth-provider-arg=client-id=611142c7-639e-47d5-b298-4d5e3520dc73\
--auth-provider-arg=tenant-id=a01c77f7-d822-4d88-88fd-9a738c46e3ab\
--auth-provider-arg=apiserver-id=ab037680-d81b-4690-b277-a39b45660a28

runn command to convert kubeconfig to enable azcli login.

kubelogin convert-kubeconfig -l azurecli


THis error occurs as the apiserver app was not granted access to the apiserver app registration

k3s@k3s-mgmt-01:~$ kubectl get nodes
Error: failed to get token: expected an empty error but received: AzureCLICredential: ERROR: AADSTS65001: The user or administrator has not consented to use the application with ID '04b07795-8ddb-461a-bbee-02f9e1bf7b46' named 'Microsoft Azure CLI'. Send an interactive authorization request for this user and resource.
Trace ID: 0109a86d-0852-4418-93c0-496727ec0800
Correlation ID: ee535b89-e6d7-4f83-b855-7b47e6f277c0
Timestamp: 2023-06-04 17:52:41Z
Interactive authentication is needed. Please run:
az login --scope ab037680-d81b---/.default


add the aaplication ID "'04b07795-8ddb-461a-bbee-02f9e1bf7b46' named 'Microsoft Azure CLI'" using expose API and grant permission. 



still not able to figure out the  association of the group to the kubectl commands. 

https://www.youtube.com/watch?v=giV0WHtp49A&t=32s

