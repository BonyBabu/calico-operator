# Setup mulitpass cluster

```bash
curl https://raw.githubusercontent.com/tigera/ccol1/main/control-init.yaml | multipass launch -n control -m 2048M 20.04 --cloud-init -
curl https://raw.githubusercontent.com/tigera/ccol1/main/node1-init.yaml | multipass launch -n node1 20.04 --cloud-init -
curl https://raw.githubusercontent.com/tigera/ccol1/main/node2-init.yaml | multipass launch -n node2 20.04 --cloud-init -
curl https://raw.githubusercontent.com/tigera/ccol1/main/host1-init.yaml | multipass launch -n host1 20.04 --cloud-init -
```

```bash
multipass start --all
```

```bash
multipass list
```

```bash
multipass shell host1
```

```bash
kubectl get nodes -A
```

# Installing Calico

## install the operator 

``bash
kubectl create -f https://docs.projectcalico.org/archive/v3.21/manifests/tigera-operator.yaml
```

## Installing Calico

```bash
cat <<EOF | kubectl apply -f -
apiVersion: operator.tigera.io/v1
kind: Installation
metadata:
  name: default
spec:
  calicoNetwork:
    containerIPForwarding: Enabled
    ipPools:
    - cidr: 198.19.16.0/21
      natOutgoing: Enabled
      encapsulation: None
EOF
```

## Validating the Calico installation

```bash
kubectl get tigerastatus/calico
```

```bash
NAME     AVAILABLE   PROGRESSING   DEGRADED   SINCE
calico   True        False         False      10m
```

