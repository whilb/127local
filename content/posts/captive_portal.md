# Captive Portal

While traveling this past week, I could not connect to Wi-Fi on the plane or at the hotel. More generally, I couldn't connect to any network that uses a captive portal to make you accept some silly terms and conditions.

## TLDR

If you're having trouble connecting to a captive portal, make sure that you don't have any docker networks that are eating your traffic.

## Investigation
First, I checked to see if I could resolve DNS with `nslookup`. No dice. That's weird.

Looking at the routing table started to give some clues. What the heck are all those bridges?

```
$ route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         172.20.0.1      0.0.0.0         UG    600    0        0 wlp0s20f3
169.254.0.0     0.0.0.0         255.255.0.0     U     1000   0        0 virbr0
172.17.0.0      0.0.0.0         255.255.0.0     U     0      0        0 docker0
172.18.0.0      0.0.0.0         255.255.0.0     U     0      0        0 br-aafda0e2fb1d
172.19.0.0      0.0.0.0         255.255.0.0     U     0      0        0 br-c6be78017438
172.20.0.0      0.0.0.0         255.255.240.0   U     600    0        0 wlp0s20f3
172.20.0.0      0.0.0.0         255.255.0.0     U     0      0        0 br-9f4674c5b183
172.24.0.0      0.0.0.0         255.255.0.0     U     0      0        0 br-c7a5ca224fa9
172.26.0.0      0.0.0.0         255.255.0.0     U     0      0        0 br-612b712247c8
192.168.122.0   0.0.0.0         255.255.255.0   U     0      0        0 virbr0
```

Even though I killed the kind-control-plane docker container, these routes still existed.
