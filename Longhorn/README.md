# Storage
<br>

There are various storage options for our storage needs:

### Options:
<ul>Rook + Ceph </ul>
<ul>Longhorn – Native Kubernetes storage. Recommended as the best viable solution. </ul>
<ul>GlusterFS + Heketi </ul>

<ul>NFS – Works well. You can create claims and manage it from Kubernetes. However, this is not clustered, and this single point of failure turns it against the exercise we are trying to do here. Secondly, very much limited to your network / connection </ul>

<br>

## Longhorn
<br>
<p><strong>Longhorn</strong> is a lightweight, reliable, and powerful distributed <a href=https://cloudacademy.com/blog/object-storage-block-storage/>block storage</a> system for Kubernetes.</p>
<p>Longhorn implements distributed block storage using containers and microservices. Longhorn creates a dedicated storage controller for each block device volume and synchronously replicates the volume across multiple replicas stored on multiple nodes. The storage controller and replicas are themselves orchestrated using Kubernetes.</p>

**Features:**
<ul>
<li>Enterprise-grade distributed block storage with no single point of failure</li>
<li>Incremental snapshot of block storage</li>
<li>Backup to secondary storage (NFS or S3-compatible object storage) built on efficient change block detection</li>
<li>Recurring snapshots and backups</li>
<li>Automated, non-disruptive upgrades. You can upgrade the entire Longhorn software stack without disrupting running storage volumes.</li>
<li>An intuitive GUI dashboard</li>
</ul>

<br>

#### Installing Longhorn

Installing of Longhorn, we can simply use helm to install it.
<br>

*On the control node:*
```
helm repo add longhorn https://charts.longhorn.io
```
```
helm repo update
# if you do not want to create separate service file for UI access as I did leter on with `service.yaml` you can use it like this:
helm install longhorn longhorn/longhorn --namespace longhorn-system --create-namespace --set defaultSettings.defaultDataPath="/storage01" --set service.ui.loadBalancerIP="10.0.1.111" --set service.ui.type="LoadBalancer"
```

<li>--set defaultSettings.defaultDataPath is optional, you can remove/edit the disks later on from UI. </li>

Look also at services. Longhorn-frontend is a management UI for storage, similar to what Rook + Ceph have. Very useful!
<br>
From the earlier commands - The UI IP address is already assigned from tag: ** --set service.ui.loadBalancerIP="10.0.1.111" **
<br>

If the IP address (tag) was not included in the Helm installation command - Create file **service.yml** with the following contents:

```
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Service
metadata:
  name: longhorn-ingress-lb
  namespace: longhorn-system
spec:
  selector:
    app: longhorn-ui
  type: LoadBalancer
  loadBalancerIP: 10.0.1.111
  ports:
    - name: http
      protocol: TCP
      port: 80
      targetPort: http
EOF

```	  

In order to apply the service.yml file:
```
kubectl apply -f service.yaml
```
<br>

```
root@control01:~# kubectl get svc --all-namespaces | grep longhorn-ingress-lb
longhorn-system   longhorn-ingress-lb        LoadBalancer   10.43.238.251   10.0.1.111   80:30405/TCP
```


#### Make Longhorn the default StorageClass
Almost done! I'd like to make Longhorn the default storage provider, and this should already be the case if you followed my guide from start, so that when using Helm (which already has a pre-set chart to use default storage provider) it will choose Longhorn.
<br>
By default, it would look like this after fresh k3s install.
<br>

```
kubectl get storageclass
NAME                 PROVISIONER             RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
local-path (default)          rancher.io/local-path   Delete          WaitForFirstConsumer   false                  6d1h
longhorn (default)   driver.longhorn.io      Delete          Immediate              true                   6d1h
```

<br>

Execute:
```
kubectl patch storageclass longhorn -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"false"}}}'
```

<br>
<p>
Sometimes the local-path storage class keeps coming back after a reboot of the host, the solution seems to be edit <strong> /var/lib/rancher/k3s/server/manifests/local-storage.yaml </strong> and look for the <strong>storageclass.kubernetes.io/is-default-class:</strong> "true" section in the StorageClass definition and comment it out.
</p>

#### Back Up Longhorn volumes

##### Set up NFS Backupstore
For using NFS server as backupstore, NFS server must support NFSv4.
<br>
The target URL should look like this:
<br>
>nfs://10.0.1.10:/backup/longhorn

<p>
Backups in Longhorn are objects in an off-cluster backupstore. A backup of a snapshot is copied to the backupstore, and the endpoint to access the backupstore is the backup target.
</p>

##### To create a backup,
<ol>
<li>Navigate to the Volume menu.</li>
<li>Select the volume you wish to back up.</li>
<li>Click Create Backup.</li>
<li>Add any appropriate labels and click OK.</li>
</ol>
