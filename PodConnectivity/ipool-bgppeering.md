# ip pools

- They are used by calico to allocate ips to resources like pods, we can annotate a namespace with ip-pool name all the resources created in that namespace will be using the ips in that pool
- We can configure ip-pool to be an overlay netwrok
- NAT can be turned-on for the ip addresses (when you use the overlay network or when traffic is routed through node ip)


```bash
kubectl cluster-info dump | grep -m 2 -E "service-cidr|cluster-cidr"
```

```bash
"k3s.io/node-args": "[\"server\",\"--flannel-backend\",\"none\",\"--cluster-cidr\",\"198.19.16.0/20\",\"--service-cidr\",\"198.19.32.0/20\",\"--write-kubeconfig-mode\",\"664\",\"--disable-network-policy\"]",
```

```bash
calicoctl get ippools
```

```bash
NAME                  CIDR             SELECTOR
default-ipv4-ippool   198.19.16.0/21   all()
```


Create an ippool
```
cat <<EOF | calicoctl apply -f - 
---
apiVersion: projectcalico.org/v3
kind: IPPool
metadata:
  name: external-pool
spec:
  cidr: 198.19.24.0/21
  blockSize: 29
  ipipMode: Never
  natOutgoing: true
  nodeSelector: "!all()"
EOF
```

```
calicoctl get ippools
```

```
Successfully applied 1 'IPPool' resource(s)
NAME                  CIDR             SELECTOR
default-ipv4-ippool   198.19.16.0/21   all()
external-pool         198.19.24.0/21   !all()
```


In this cluster Calico has been configured to allocate IP addresses for pods from the 198.19.16.0/21 CIDR (which is a subset of the cluster pod CIDR, 198.19.16.0/20, configured on Kubernetes).

Overall we have the following address ranges:

198.19.16.0/20 - Cluster Pod CIDR
198.19.16.0/21- Default IP Pool CIDR
198.19.32.0/20 - Service CIDR

# bgp peering
bgp peering is used to share the routes between nodes and routes, in order to reduce the number of routes it uses ippool block sizes in the route table.

<img width="1057" height="602" alt="image" src="https://github.com/user-attachments/assets/cedb12a0-666d-41b1-9fff-8dde5509124e" />



