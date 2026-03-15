# Cluster IP
(src: pod1, dest: cluster-ip) -> (kube-proxy - DNAT) -> (src: pod1, dest: pod2)

## Read IP table

The rules for the summary service(cluster ip backed with 2 pods and 2 endpoints).

```bash
sudo iptables -v --numeric --table nat --list KUBE-SERVICES | grep -E summary
```

The KUBE-SERVICES chain matches on the clusterIP and jumps to the corresponding KUBE-SVC-XXXXXXXXXXXXXXXX chain.

The KUBE-SVC-XXXXXXXXXXXXXXXX chain load balances the packet to a random service endpoint KUBE-SEP-XXXXXXXXXXXXXXXX chain.

The KUBE-SEP-XXXXXXXXXXXXXXXX chain DNATs the packet so it will get routed to the service endpoint (backing pod).

Finally, let's return to host1 to take a look at NodePorts in the next section by using the exit command.

# Node Port
(src: client, dst: ingress-controller) -> (kube-proxy - NAT)(nodeport -> cluster-ip use dnat to get po) -> (src: ingress-controller, dst: pod) ->  
(src: pod, dst: ingress-controller) -> (kube-proxy - NAT) -> (src: ingress-controller, dst: client)

```bash
sudo iptables -v --numeric --table nat --list KUBE-SERVICES | grep KUBE-NODEPORTS
```

The end of the KUBE-SERVICES chain jumps to the KUBE-NODEPORTS chain

The KUBE-NODEPORTS chain matches on the NodePort and jumps to the corresponding KUBE-SVC-XXXXXXXXXXXXXXXX chain.
The KUBE-SVC-XXXXXXXXXXXXXXXX chain load balances the packet to a random service endpoint KUBE-SEP-XXXXXXXXXXXXXXXX chain.
The KUBE-SEP-XXXXXXXXXXXXXXXX chain DNATs the packet so it will get routed to the service endpoint (backing pod).
