# raspberry-pi-ansible

## Network Topology

```
ISP --> Router [192.168.10.1/24] --> pi-31 [192.168.10.31/24] Wi-Fi ))  (( Wi-Fi Router/Switch --> pi-41...pi-44
```

- Router (`192.168.10.1/24`) provides internet via 5GHz Wi-Fi for our devices.
- pi-31 is connected to the router via ethernet and provides an additional 2.4GHz access point for the Pi cluster.
- Pi cluster connects to pi-31 via Wi-Fi switch (a old 2.4GHz router, that works in Wi-Fi client-mode). All Pis are
  connected to the switch via ethernet.
- Pi cluster lives in a separate network space `192.168.20.1/24`, provided by `wlan0` on pi-31.

## Routing

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
