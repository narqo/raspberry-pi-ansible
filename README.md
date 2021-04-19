# raspberry-pi-ansible

## Network Topology

```
ISP
 ↳ Router [192.168.10.1/24] Wi-Fi ~)) ((~ pi-31 [192.168.10.31/24]
                                            🡙
                                          Switch
                                            🡙
                                      pi-41...pi-44 [192.168.20.41..44/24]
```

- Router (`192.168.10.1/24`) provides internet via 5GHz Wi-Fi for home devices.
- `pi-31` connects to the router via Wi-Fi.
- Pi cluster connects to `pi-31` though a network switch.
- Pi cluster lives in a separate IPv4 network space `192.168.20.1/24` (using static IPs `192.168.20.41...192.168.20.44`) and configured to access the
  Routers network via NAT.
- Pi cluster lives in IPv6 sub-network, assigned by the ISP:
  - the ISP provides `2001:db8:abc:1::/64`;
  - Pi-nodes are assigned static IPv6 `2001:db8:abc:1:41::1...2001:db8:abc:1:44::1`;
  - `ndppd` on `pi-31` proxies NDP (neighbour discovery protocol) from `2001:db8:abc:1:40::/76` on `wlan0` to `eth0`, allowing both Pi cluster to access
    the IPv6 internet, and the devices on the home network to access the Pi cluster via IPv6 w/o the NAT.
- `avahi-daemon` on `pi-31` is set to reflect incoming mDNS requests to all local networks (`wlan0` and `eth0`), allowing home devices to discover the nodes of Pi cluster under `.local` domain.

## Routing (IPv4)

Commands below add a static route via gateway `pi-31` (`192.168.10.31/24`), allowing the host to connect to pi cluster
(`192.168.20.0/24`) from the main router's network.

### Adding static route on macOS

```
$ sudo route -n add 192.168.20.0/24 192.168.10.31

$ netstat -nr | grep 192.168.
default            192.168.10.1       UGSc           en0
···
192.168.20         192.168.10.31      UGSc           en0
```

To delete the route, run `sudo route -n delete 192.168.20.0/24 192.168.10.31`.

### Adding static route on Linux

```
$ sudo ip route add 192.168.20.0/24 via 192.168.10.31

$ sudo ip r s
default via 192.168.10.1 dev eth0 src 192.168.10.31 metric 202
···
192.168.20.0/24 dev wlan0 proto dhcp scope link src 192.168.20.1 metric 303
```

To delete the route, run `sudo ip r del 192.168.20.0/24`.

## Routing (IPv6)

Add Pi cluster's IPv6 subnet to the routing table on pi-31:

```
$ sudo ip r add 2001:db8:abc:1:40::/76 dev eth0 metric 100
```

Above, `metric 100` is required to make sure the route had a bigger priority than the `/64` route set from the router.

Thanks to the fact, that each Pi has a globally routable address and the NDP proxying, the cluster is accessible w/o NAT or additional routing.

## Ansible Playbook

### k3s

```
$ ansible-playbook -i hosts --limit=pi_kube_servers -e k3s_version=v1.19.4+k3s2 playbooks/k3s.yml
```
