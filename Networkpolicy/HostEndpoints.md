# Host hardening:

Calico global network policies can be used to manage the traffic into the host(including external hosts).


## Create Host Endpoint

- Verify no host endpoints exist:

```bash
calicoctl get heps
```

- To configure a host endpoint for every node in the cluster update the kubcontrollerconfig manifest:

```bash
calicoctl patch kubecontrollersconfiguration default --patch='{"spec": {"controllers": {"node": {"hostEndpoint": {"autoCreate": "Enabled"}}}}}'
```


Create a global network policy:

```bash
cat <<EOF| calicoctl apply -f -
---
apiVersion: projectcalico.org/v3
kind: GlobalNetworkPolicy
metadata:
  name: default-node-policy
spec:
  selector: has(kubernetes.io/hostname)
  ingress:
  - action: Allow
    protocol: TCP
    source:
      nets:
      - 127.0.0.1/32
  - action: Allow
    protocol: UDP
    source:
      nets:
      - 127.0.0.1/32
EOF
```

It allows all TCP/UDP traffic between pods in the host, but block all external traffic

Note: Use nc to test on port 21 of any nodes

*** Calico has a list of configurable fail safe ports which takes precedence over any other policies. It allow all the connection used to keep the k8s control pland and calico control plane up and running
It also allow ssh port on all the host on k8s cluster.***

## Lockdown Nodeport access

kubeproxy forward traffic to nodeport which then forwared to a pod at the backend of a service. the mapping from nodeport to pod address is done using DNAT(Destination Network address translation). it is possible block this traffic prior to this translation happens(i.e a policy which sees nodeport as a destination not the backend pod)

```bash
cat <<EOF | calicoctl apply -f -
---
apiVersion: projectcalico.org/v3
kind: GlobalNetworkPolicy
metadata:
  name: nodeport-policy
spec:
  order: 100
  selector: has(kubernetes.io/hostname)
  applyOnForward: true
  preDNAT: true
  ingress:
  - action: Allow
    protocol: TCP
    destination:
      ports: [30180]
    source:
      nets:
      - 198.19.15.254/32
  - action: Deny
    protocol: TCP
    destination:
      ports: ["30000:32767"]
  - action: Deny
    protocol: UDP
    destination:
      ports: ["30000:32767"] # 30000:32767 ports allocated to the services
EOF
```

Only traffic from 198.19.15.254/32 is allowed to connect to node-port at the port 30180







