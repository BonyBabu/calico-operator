EBPF dataplane:
- No NAT
- Client ip is preserved as source till pod
- Create network policies based on the external ip
- Possible to clean up the iptables via felix configuration once ebpf is enabled

DSR:
- Return traffic is can reach the client directly from Node where pod is deployed
- Network needs to configured to accept traffic from the non requested ips
