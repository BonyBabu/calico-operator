# WIREGUARD

<img width="1010" height="535" alt="image" src="https://github.com/user-attachments/assets/c5806fbc-34d9-4390-aa29-ee2dd1abdaa8" />

## how to enable wireguard
```bash
calicoctl patch felixconfiguration default --type='merge' -p '{"spec":{"wireguardEnabled":true}}'
```

## public key used to encrypt 
```
calicoctl get node node1 -o yaml

apiVersion: projectcalico.org/v3
kind: Node
metadata:
  annotations:
    projectcalico.org/kube-labels: '{"beta.kubernetes.io/arch":"amd64","beta.kubernetes.io/instance-type":"k3s","beta.kubernetes.io/os":"linux","k3s.io/hostname":"node1","k3s.io/internal-ip":"198.19.0.2","kubernetes.io/arch":"amd64","kubernetes.io/hostname":"node1","kubernetes.io/os":"linux","node.kubernetes.io/instance-type":"k3s"}'
  creationTimestamp: "2020-10-20T23:33:09Z"
  labels:
    beta.kubernetes.io/arch: amd64
    beta.kubernetes.io/instance-type: k3s
    beta.kubernetes.io/os: linux
    k3s.io/hostname: node1
    k3s.io/internal-ip: 198.19.0.2
    kubernetes.io/arch: amd64
    kubernetes.io/hostname: node1
    kubernetes.io/os: linux
    node.kubernetes.io/instance-type: k3s
  name: node1
  resourceVersion: "8760"
  uid: 66f16def-8f76-46dc-90e3-491bfc75dc9b
spec:
  bgp:
    ipv4Address: 198.19.0.2/20
    ipv4IPIPTunnelAddr: 198.19.22.128
  orchRefs:
  - nodeName: node1
    orchestrator: k8s
  wireguard:
    interfaceIPv4Address: 198.19.22.131
status:
  podCIDRs:
  - 198.19.17.0/24
  wireguardPublicKey: An4UT4PR9XzGgJ5df452Dw034q1SfXZOI1Dp2ebUUWQ=
```

## ssh into node
```bash
ip addr | grep wireguard
```

```
10: wireguard.cali: <POINTOPOINT,NOARP,UP,LOWER_UP> mtu 1400 qdisc noqueue state UNKNOWN group default qlen 1000
    inet 198.19.22.131/32 brd 198.19.22.131 scope global wireguard.cali
```



