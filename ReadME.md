# k3s
**Collection of all Kubernetes (k3s) resources - Charts, secrets, including initial setup/config**
<img src='https://learn.g2.com/hubfs/Kubernetes%20architecture.png'/> <br>
*Image Source: G2 Learn Hub

# Initial Setup of hosts

## Kubernetes Install

### This is our primary node: Master01
We are going to install the K3s version of Kubernetes, that is lightweight enough for out single board computers to handle. Use the following command to download and initialize K3s’ master node:
```
curl -sfL https://get.k3s.io | sh -s - --write-kubeconfig-mode 644 --disable servicelb --token some_random_password --node-taint CriticalAddonsOnly=true:NoExecute --bind-address 10.0.1.100 --disable-cloud-controller --disable local-storage
```

Quick summary of the tags above:<br>

_--write-kubeconfig-mode 644 -  (This is the mode that we want to use for the kubeconfig file.)_ <br>
_--disable servicelb -  (This is the flag that we want to use to disable the service load balancer. (We will use metallb instead)_ <br>
_--token -  (This is the token that we want to use to connect to the K3s master node. Choose a random password, but keep remember it.)_ <br>
_--node-taint -  (This is the flag that we want to use to add a taint to the K3s master node. Marks node to run only critical pods.)_ <br>
_--bind-address -  (This is the flag that we want to use to bind the K3s master node to a specific IP address.)_ <br>
_--disable-cloud-controller -  (This is the flag that we want to use to disable the K3s cloud controller. Not needed if deployed locally.)_ <br>
_--disable local-storage -  (This is the flag that we want to use to disable the K3s local storage. Not needed - Longhorn (or NFS) will be used.)_ <br>

<br>

### This is our worker node/s: worker01
```
curl -sfL https://get.k3s.io | K3S_URL=https://10.0.1.100:6443 K3S_TOKEN=some_random_password sh -
```

<br>

Setting role/labels (We can tag our cluster nodes to give them labels)
K3s lets pods run on the control plane. You can add a taint on the master node when initial install of K3s. For more control over where workloads are deployed - you can use labels to specify where pods and deployments should run.

Let's add this tag key:value: kubernetes.io/role=worker to worker nodes. This is more cosmetic, to have nice output from kubectl get nodes.
```
kubectl label nodes worker01 kubernetes.io/role=worker
```

Another label/tag. I will use this one to tell deployments to prefer nodes where node-type equals workers. The node-type is our chosen name for key, you can call it whatever.
```
kubectl label nodes worker01 node-type=worker
```

<br>

### Connect remotely to the cluster
If you don’t want to connect via SSH to a node every time you need to query your cluster, it is possible to install kubectl (k8s command line tool) on your local machine and control remotely your cluster.<br>


Install kubectl on your local machine
Install kubectl binary with curl on Linux
<ol>
<li> Download the latest release with the command: </li>
  
```  
  curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl" 
```

  
<li> Install kubectl: </li>

```

sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

```

  
<li> Copy the k3s config file from the master node to your local machine: </li>

 > We simply need to download the file /etc/rancher/k3s/k3s.yaml located on the master node to our local machine into ~/.kube/config.

</ol>

### OPTIONAL: Kubernetes Dashboard
Kubernetes Dashboard is a web-based user interface (UI) for managing and monitoring Kubernetes clusters. It provides a graphical representation of the Kubernetes resources and allows users to interact with their clusters without needing to use the command-line interface (CLI) tool kubectl for every operation.

#### Installing Kubernetes-dashboard using manifest file.

##### Step 1: Deploy the Dashboard
First, you need to deploy the Kubernetes Dashboard using its official YAML file.

````
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.3.1/aio/deploy/recommended.yaml
````

<p>This command deploys the Dashboard using a recommended configuration.</p>
After a few seconds, you should see to pods running in the namespace kubernetes-dashboard.

```
kubectl get pods -n kubernetes-dashboard

NAME                                        READY   STATUS    RESTARTS   AGE
dashboard-metrics-scraper-b4684615-hoi6ho   1/1     Running   0          15m
kubernetes-dashboard-6345dvrl-41wliug       1/1     Running   0          15m

```

##### Step 2: Create a Service Account and Cluster Role Binding
To access the Kubernetes Dashboard, you'll need a Service Account and Cluster Role Binding. Create a YAML file (e.g., dashboard-adminuser.yaml) with the following content:

```
cat <<EOF | kubectl apply -f -
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kubernetes-dashboard
---
EOF
```

##### Step 3: Authenticate with the Dashboard
When you access the Dashboard URL, you will be prompted for authentication. To obtain an authentication token, run the following command:
```
kubectl -n kubernetes-dashboard describe secret $(kubectl -n kubernetes-dashboard get secret | grep admin-user-token | awk '{print $1}')
```

##### Step 4: Access the Kubernetes Dashboard
To access the Dashboard, you can use the kubectl proxy command, which creates a secure channel to the Dashboard:

```
kubectl proxy
```
Copy the token and paste it into the Dashboard login page to log in.

<p>The Kubernetes Dashboard will be available at this URL:<br>
  <a target="_blank" rel="noopener" href="http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/">http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/</a>.</p>

> Note: When accessing the Dashboard in a production environment, it's recommended to use a more secure method like SSH tunneling or set up an Ingress Controller with proper security configurations.


<img src='https://www.aquasec.com/wp-content/uploads/2023/05/3.png'/> *Image source: Kubernetes*

##### Step 5: Explore the Kubernetes Dashboard
<p>You should now have access to the Kubernetes Dashboard. You can explore and manage your cluster resources through the web interface.</p>

<img src='https://www.aquasec.com/wp-content/uploads/2023/05/4.png'/> *Image source: Kubernetes*

<p>Please be cautious when exposing the Kubernetes Dashboard to the public internet, and consider securing it properly in production environments. Additionally, keep your Kubernetes cluster up-to-date and regularly review the Kubernetes Dashboard documentation for any changes or updates.</p>
