# MetalLB - Kubernetes Load Balancer
#### https://metallb.universe.tf/
<br>
MetalLB is a load-balancer implementation for bare metal Kubernetes clusters. When configuring a Kubernetes service of type LoadBalancer, MetalLB will dedicate a virtual IP from an address-pool to be used as load balancer for an application.

## Installation:

```
# First add metallb repository to your helm
helm repo add metallb https://metallb.github.io/metallb
# Check if it was found
helm search repo metallb
# Install metallb
helm upgrade --install metallb metallb/metallb --create-namespace \
--namespace metallb-system --wait
```

If Metallb is installed succesffully, you should get the following message: 
>MetalLB is now running in the cluster.

Now we need to configure it.

## Configuration

MetalLB needs to know what IP range to use for external IPs. This is done via a Custom Resource. You can find the CR in the official documentation. So lets create and apply a Custom Resource with the following content:
<br>

```
cat << 'EOF' | kubectl apply -f -
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: default-pool
  namespace: metallb-system
spec:
  addresses:
  - 10.0.1.110-10.0.1.130
---
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: default
  namespace: metallb-system
spec:
  ipAddressPools:
  - default-pool
EOF
```
Once configured correctly - you should get the following message:
>ipaddresspool.metallb.io/default-pool created
<br>l2advertisement.metallb.io/default created

Done! Now every time a new Kubenertes service of type **LoadBalancer** is deployed, MetalLB will assign an IP from the pool to access the application. In this case - IP addresses will be assigned from the pool: **10.0.1.110 - 10.0.1.130**
