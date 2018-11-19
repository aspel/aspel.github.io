---
layout: post
title: "OSPF over GRE tunnel with IPSec (Mikrotik and PFsense) and two ISP"
date: "2018-11-19 14:26:38 +0200"
categories: blog
location: Antarctica, Troll
description: This is simple manual about how to setup failover channel between Mikrotik and PFsense
---
---
It's a simple manual how to setup failover channel between **Mikrotik** and **PFsense**.

![My screenshot]({{ site.url }}/assets/ipsec-kiev-fra.svg)

Mikrotik network settings:
 * Local: 10.1.1.0/24
 * ISP1: 1.1.1.1/32
 * ISP2: 2.2.2.2/32
 * gre0: 10.160.254.49/30
 * gre1: 10.160.254.53/30

PFsense network settings:
 * Local: 10.3.3.0/24
 * ISP0: 3.3.3.3/32
 * gre0: 10.160.254.50/30
 * gre1: 10.160.254.54/30

## Configure Mikrotik
```bash
/ip ipsec peer profile add name="ph1" hash-algorithm=sha1 enc-algorithm=aes-256 dh-group=modp1024 lifetime=1d proposal-check=obey nat-traversal=no dpd-interval=10s dpd-maximum-failures=5
/ip ipsec proposal add name="ph2" auth-algorithms=sha1,md5 enc-algorithms=aes-256-cbc lifetime=30m pfs-group=modp1024
/ip ipsec peer add address=3.3.3.3/32 local-address=1.1.1.1 profile=ph1 auth-method=pre-shared-key secret=XXXXXX generate-policy=no policy-template-group=default exchange-mode=ike2 notrack-chain="output" send-initial-contact=yes comment="IPSO -> IPS1" my-id=address:1.1.1.1
/ip ipsec peer add address=3.3.3.3/32 local-address=2.2.2.2 profile=ph1 auth-method=pre-shared-key secret=XXXXXX generate-policy=no policy-template-group=default exchange-mode=ike2 notrack-chain="output" send-initial-contact=yes comment="IPSO -> IPS2" my-id=address:2.2.2.2
/ip ipsec policy add src-address=1.1.1.1/32 src-port=any dst-address=3.3.3.3/32 dst-port=any protocol=all action=encrypt level=unique ipsec-protocols=esp tunnel=no proposal=ph2 comment="ISP0 -> ISP1"
/ip ipsec policy add src-address=2.2.2.2/32 src-port=any dst-address=3.3.3.3/32 dst-port=any protocol=all action=encrypt level=unique ipsec-protocols=esp tunnel=no proposal=ph2 comment="ISP0 -> ISP2"


/interface gre add name="gre-tunnel-isp0-isp1" mtu=1400 local-address=1.1.1.1 remote-address=3.3.3.3 keepalive=10s,10 dscp=inherit clamp-tcp-mss=yes dont-fragment=no allow-fast-path=no
/interface gre add name="gre-tunnel-isp0-isp2" mtu=1400 local-address=2.2.2.2 remote-address=3.3.3.3 keepalive=10s,10 dscp=inherit clamp-tcp-mss=yes dont-fragment=no allow-fast-path=no

/ip address add address=10.160.254.49/30 interface=gre-tunnel-isp0-isp1  comment="GRE-ISP1"
/ip address add address=10.160.254.53/30 interface=gre-tunnel-isp0-isp2  comment="GRE-ISP2"

/routing ospf instance add name="ospf1" router-id=10.1.1.1 distribute-default=never redistribute-connected=no redistribute-static=no redistribute-rip=no redistribute-bgp=no redistribute-other-ospf=no metric-defaul
t=1 metric-connected=20 metric-static=20 metric-bgp=auto metric-other-ospf=auto in-filter=ospf-in out-filter=ospf-out
/routing ospf area add area-id=1.1.1.1 instance=ospf1 name=area1
/routing ospf network add network=10.160.254.48/30 area=area1
/routing ospf network add network=10.160.254.52/30 area=area1

```

## Configure PFsense
#### IPsec
VPN -> IPsec -> Add P1

![vpn ph1]({{ site.url }}/assets/VPN IPsec Tunnels Edit Phase 1.png)

VPN -> IPsec -> ISP1 -> Add P2

![vpn ph2]({{ site.url }}/assets/VPN IPsec Tunnels Edit Phase 2.png)

The same settings for ISP2

VPN -> IPsec -> Add P1

VPN -> IPsec -> ISP2 -> Add P2


#### GRE
Interfaces -> GREs -> Add

![gre]({{ site.url }}/assets/Interfaces GREs Edit.png)

The same settings for ISP2

Interfaces -> Interface Assignments

ADD OPT1, OPT2

Then please select OPT1 and OPT2 and enable them

![gres]({{ site.url }}/assets/InterfacesOPT1.png)


#### Firewall
Firewall->NAT->Outbound

Select **Manual Outbound NAT**

And delete all auto-mappings

![NAT]({{ site.url }}/assets/Firewall NAT Outbound.png)


Firewall->Rules->Floating

Create a new Rule:
 * Any to Any
 * TCP Flags: Any flags
 * State type: None

![Float]({{ site.url }}/assets/Firewall Rules Floating Edit.png)


#### OSPF

System -> Package Manager -> Available Packages

search **Quagga_OSPF** and install it

Services -> Quagga OSPFd -> Interface Settings

Add two Interfaces

![Quagga1]({{ site.url }}/assets/Services Quagga OSPFd Interface Settings.png)


Then edit global settings

![Quagga2]({{ site.url }}/assets/Services Quagga OSPFd Global Settings.png)
