

Block Egress and ingress by default

```bash
cat <<EOF | calicoctl apply -f -
apiVersion: projectcalico.org/v3
kind: GlobalNetworkPolicy
metadata:
  name: default-app-policy
spec:
  namespaceSelector: has(projectcalico.org/name) && projectcalico.org/name not in {"kube-system", "calico-system", "calico-apiserver"}
  types:
  - Ingress
  - Egress
EOF
```

Only workloads deployed in namespaces "kube-system", "calico-system", "calico-apiserver" are allowed to have ingress and egress traffic



## Block Internet by default

```bash
cat <<EOF | calicoctl apply -f -
apiVersion: projectcalico.org/v3
kind: GlobalNetworkPolicy
metadata:
  name: egress-lockdown
spec:
  order: 600
  namespaceSelector: has(projectcalico.org/name) && projectcalico.org/name not in {"kube-system", "calico-system"}
  serviceAccountSelector: internet-egress not in {"allowed"}
  types:
  - Egress
  egress:
    - action: Deny
      destination:
        notNets:
          - 10.0.0.0/8
          - 172.16.0.0/12
          - 192.168.0.0/16
          - 198.18.0.0/15
EOF
```

The networks referenced in the manifest above are defined by RFC5735, which is a superset of RFC1918.
Policy order 600 gets precedence over policy order 1000 (kubernetes network policy precedence), means k8s np cannot overide this one


Only secops team has the RBAC permissions to create namespaces, service accounts, Calico Global Network Policies. Similarly, tenants have the RBAC permissions to deploy pods and K8 Network policies in the namespace allocated to them. By creating Global Network policies based using labels assigned for namespaces and service-accounts, secops team can automate the security changes without impacting CI/CD of indvidual developers.

shift-left security practices (i.e. delegating different levels of security trust to dev or other teams)
