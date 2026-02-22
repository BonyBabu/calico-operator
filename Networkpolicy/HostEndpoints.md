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

Note: Use nc to test on port 21

*** Calico has a list of configurable fail safe ports which takes precedence over any other policies. It allow all the connection used to keep the k8s control pland and calico control plane up and running
It also allow ssh port on all the host on k8s cluster.***
