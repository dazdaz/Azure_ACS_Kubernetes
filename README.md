<pre>
# Deployed on 2nd Sep 2017
# Deploys Ubuntu 16.04.02, Docker 1.12.6, Kubernetes 1.6.6

RESOURCE_GROUP=washk8srg
DNS_PREFIX=washk8sweb
CLUSTER_NAME=k8s-acs-wash
LOCATION=southeastasia

$ az acs create --orchestrator-type=kubernetes --resource-group $RESOURCE_GROUP --name=$CLUSTER_NAME --dns-prefix=$DNS_PREFIX --admin-username=azureuser --admin-password secret123 --generate-ssh-keys --master-count=1 --agent-count=3 --agent-vm-size=Standard_D1_v2
$ az acs list --resource-group $RESOURCE_GROUP --output jsonc
$ az acs kubernetes get-credentials --name $CLUSTER_NAME --resource-group $RESOURCE_GROUP
$ sudo az acs kubernetes install-cli

$ ls -la .kube/config
-rw------- 1 chilli chilli 6337 Sep  2 20:05 .kube/config

$ kubectl get nodes -o wide
NAME                    STATUS                     AGE       VERSION   EXTERNAL-IP   OS-IMAGE                      KERNEL-VERSION
k8s-agent-519f5f2c-0    Ready                      1m        v1.6.6    <none>        Debian GNU/Linux 8 (jessie)   4.4.0-93-generic
k8s-agent-519f5f2c-1    Ready                      1m        v1.6.6    <none>        Debian GNU/Linux 8 (jessie)   4.4.0-93-generic
k8s-agent-519f5f2c-2    Ready                      1m        v1.6.6    <none>        Debian GNU/Linux 8 (jessie)   4.4.0-93-generic
k8s-master-519f5f2c-0   Ready,SchedulingDisabled   1m        v1.6.6    <none>        Debian GNU/Linux 8 (jessie)   4.4.0-93-generic

# View the worker nodes InternalIP Addresses
kubectl get nodes --output=jsonpath='{range .items[*]}{.status.addresses[?(@.type=="InternalIP")].address} {.spec.podCIDR} {"\n"}{end}'

# Copy private RSA key onto k8s Master
scp ~/.ssh/id_rsa azureuser@washk8swebmgmt.southeastasia.cloudapp.azure.com:~/.ssh/

# Test SSH onto k8s Master
$ ssh -l azureuser washk8swebmgmt.southeastasia.cloudapp.azure.com
# Use ProxyJump introduced into OpenSSH 7.3, to send commands directly to the k8s agents
$ ssh -A -J azureuser@washk8swebmgmt.southeastasia.cloudapp.azure.com azureuser@node1 "sudo apt-get update"
$ ssh -A -J azureuser@washk8swebmgmt.southeastasia.cloudapp.azure.com azureuser@node2 "sudo apt-get update"
$ ssh -A -J azureuser@washk8swebmgmt.southeastasia.cloudapp.azure.com azureuser@node3 "sudo apt-get update"

# Install helm 2.5.1 - Compatible with aci-connector
$ wget curl https://kubernetes-helm.storage.googleapis.com/helm-v2.5.1-linux-amd64.tar.gz
$ sudo tar xvzf helm-v2.5.1-linux-amd64.tar.gz -C /usr/local/bin/ --strip-components=1 linux-amd64/helm

# Install helm 2.6.1
$ curl https://raw.githubusercontent.com/kubernetes/helm/master/scripts/get > get_helm.sh
$ chmod 700 get_helm.sh
$ ./get_helm.sh

$ helm init
</pre>

# Install & Configure aci-connector-k8s
https://github.com/Azure/aci-connector-k8s
