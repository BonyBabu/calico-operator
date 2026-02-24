# POD Connectivity

Pod traffic is tunnelled through underlying network of host


<img width="2011" height="1159" alt="image" src="https://github.com/user-attachments/assets/86c9efb3-8ee5-489f-9a28-a68da2c7e2bc" />


It is possible to hide the pod ip using overlay mode vxlan/ipip

## Exec into the pod

```bash
CUSTOMER_POD=$(kubectl get pods -n yaobank -l app=customer -o name)
kubectl exec -ti -n yaobank $CUSTOMER_POD -- /bin/bash
```

### Interfaces in pods
```bash
ip addr
```

```bash
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
3: eth0@if9: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1410 qdisc noqueue state UP group default
    link/ether 3a:0c:14:d0:92:89 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 198.19.22.132/32 brd 198.19.22.132 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::380c:14ff:fed0:9289/64 scope link
       valid_lft forever preferred_lft forever
```

- 1: loopback interface for localhost
- 2: eth0 interfect with the pods ip address

### Details of the network interfaces 

```bash
ip -c link show up
```

```bash
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
3: eth0@if9: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1410 qdisc noqueue state UP mode DEFAULT group default
    link/ether 3a:0c:14:d0:92:89 brd ff:ff:ff:ff:ff:ff link-netnsid 0
```

eth0 is the network interface of the pod to the host network namespace(represented by link-netnsid 0) on the pod side, if9 represent the pods network interface on the host side

### Route tables in pods

```bash
ip route
```

```bash
default via 169.254.1.1 dev eth0
169.254.1.1 dev eth0  scope link 
```

Pods default route is out over the eth0 interface: 169.254.1.1
Next hop address of 169.254.1.1 is a dummy address used by Calico



## ssh into node where the pod is deployed
look for the network interface 90
```bash
ip -c link show up
```

```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP mode DEFAULT group default qlen 1000
    link/ether 00:15:5d:60:a5:a3 brd ff:ff:ff:ff:ff:ff
5: cali1eaab2bfc77@if3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1410 qdisc noqueue state UP mode DEFAULT group default
    link/ether ee:ee:ee:ee:ee:ee brd ff:ff:ff:ff:ff:ff link-netns cni-8174e7bb-f2a6-0b61-1282-2c425f949ab5
6: cali9c9ee09e807@if3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1410 qdisc noqueue state UP mode DEFAULT group default
    link/ether ee:ee:ee:ee:ee:ee brd ff:ff:ff:ff:ff:ff link-netns cni-e269d0db-f258-6f12-8f31-064bbb4cf87c
7: calid35188eb0ba@if3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1410 qdisc noqueue state UP mode DEFAULT group default
    link/ether ee:ee:ee:ee:ee:ee brd ff:ff:ff:ff:ff:ff link-netns cni-cebefddd-a569-08c2-29d7-7551967f7cf4
8: calif0a98285df9@if3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1410 qdisc noqueue state UP mode DEFAULT group default
    link/ether ee:ee:ee:ee:ee:ee brd ff:ff:ff:ff:ff:ff link-netns cni-0a89d746-b16d-2299-9e6e-948dd7b2b512
9: caliea2aa288365@if3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1410 qdisc noqueue state UP mode DEFAULT group default
    link/ether ee:ee:ee:ee:ee:ee brd ff:ff:ff:ff:ff:ff link-netns cni-f7159375-bf04-e8d8-85b8-0da49bfd53d0
```

we see caliea2aa288365 which links to @if3 in network namespace ID 3 (look at the 3rd newtor interface in the pod)
pod's if3 is connected to nodes if9 and viceversa


## How Hosts Route Pod Traffic
```bash
ubuntu@node1:~$ ip route
default via 192.168.64.1 dev ens3 proto dhcp src 192.168.64.3 metric 100
192.168.64.0/24 dev ens3 proto kernel scope link src 192.168.64.3 metric 100
192.168.64.1 dev ens3 proto dhcp scope link src 192.168.64.3 metric 100
198.19.0.0/20 dev ens3 proto kernel scope link src 198.19.0.2
198.19.21.0/26 via 198.19.0.1 dev ens3 proto bird
198.19.21.64/26 via 198.19.0.3 dev ens3 proto bird
198.19.22.128 dev cali4004ef0a9b2 scope link
blackhole 198.19.22.128/26 proto bird
198.19.22.129 dev caliebd5a32bb7c scope link
198.19.22.130 dev cali9b2f02cff11 scope link
```

- Traffic to the pod (with in the node) with ip  198.19.22.130 goes via cali9b2f02cff11
  - 198.19.22.130 dev cali9b2f02cff11 scope link
- Traffic to the pods (on other nodes) with ips in range 198.19.21.0/26 is forwarded through host's eth0
  - 198.19.21.64/26 via 198.19.0.3 dev ens3 proto bird    
- blackhole rule tells the host which ips are allocated(from calico ipam) for pods in this node, via bgp peering. if it can't find a more specific route for an individual IP in that block then it should discard the packet 






