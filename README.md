Deploy and configure network load balancer

Doc: https://kubernetes.io/docs/tasks/access-application-cluster/create-external-load-balancer/


# Deploy metalLb
## You have to enable strict ARP mode
```bash
kubectl edit configmap -n kube-system kube-proxy
```
## And set
```yaml
mode: "ipvs"
ipvs:
  strictARP: true
```

## Download and install MetalLB

```bash
git clone https://github.com/metallb/metallb.git
mkdir metallb
cd metallb
wget https://raw.githubusercontent.com/metallb/metallb/v0.13.3/config/manifests/metallb-native.yaml
```

# We are giving MetalLB an IP range from our cluster infra to allocate from
```bash
cat > metallb-config.yaml
```
```yaml
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: first-pool
  namespace: metallb-system
spec:
  addresses:
  - 172.16.1.101-172.16.1.150
---
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: advert
  namespace: metallb-system
```

# Apply the manifests
```bash
kubectl apply -f metallb-native.yaml
kubectl apply -f metallb-config.yaml
```

# Now we create the deployment with a Load Balancer service type
```bash
kubectl create deployment nginx-lb --image=nginx:latest
kubectl scale deployment nginx-lb --replicas=2
kubectl expose deployment nginx-lb --port=80 --target-port=80 --type=LoadBalancer
kubectl describe svc nginx-lb
```
Name:                     nginx-lb
Namespace:                default
Labels:                   app=nginx-lb
Annotations:              <none>
Selector:                 app=nginx-lb
Type:                     LoadBalancer
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       10.104.174.166
IPs:                      10.104.174.166
LoadBalancer Ingress:     172.16.1.101
Port:                     <unset>  80/TCP
TargetPort:               80/TCP
NodePort:                 <unset>  31264/TCP
Endpoints:                10.244.1.17:80,10.244.2.17:80
Session Affinity:         None
External Traffic Policy:  Cluster
Events:
  Type    Reason        Age   From                Message
  ----    ------        ----  ----                -------
  Normal  IPAllocated   27s   metallb-controller  Assigned IP ["172.16.1.101"]
  Normal  nodeAssigned  27s   metallb-speaker     announcing from node "k8s-node-1" with protocol "layer2"

# We are getting the page through the IP address allocated by MetalLB from the pool we provided
```bash
curl http://172.16.1.101:80
```
...
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>
