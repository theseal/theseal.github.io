---
layout: post
title:  "IPv6 on MikroTik hEX"
date:   2023-02-18
---

So [the](/2015/11/01/edgeroute-ipv6-done-right.html) [series](/2020/12/07/ipv6-on-usg.html) continues… fourth router since 2015 (no worries each of every one of past hardware has reached new home(s)!).

This time a [MikroTik hEX (RB750Gr3)](https://mikrotik.com/product/RB750Gr3) found it's way to me and needed to be configured for [IPv6](https://en.wikipedia.org/wiki/IPv6) with [Bahnhof](https://bahnhof.se) in [Sundbybergs stadsnät](https://www.sundbybergsstadsnat.se). Unlike previous hardware this was pretty straight forward.

First of all make sure that the default firewall is enabled for IPv6
```
/ipv6 firewall filter
add action=accept chain=input comment="defconf: accept established,related,untracked" connection-state=established,related,untracked
add action=drop chain=input comment="defconf: drop invalid" connection-state=invalid
add action=accept chain=input comment="defconf: accept ICMPv6" protocol=icmpv6
add action=accept chain=input comment="defconf: accept UDP traceroute" port=33434-33534 protocol=udp
add action=accept chain=input comment="defconf: accept DHCPv6-Client prefix delegation." dst-port=546 protocol=udp src-address=fe80::/10
add action=accept chain=input comment="defconf: accept IKE" dst-port=500,4500 protocol=udp
add action=accept chain=input comment="defconf: accept ipsec AH" protocol=ipsec-ah
add action=accept chain=input comment="defconf: accept ipsec ESP" protocol=ipsec-esp
add action=accept chain=input comment="defconf: accept all that matches ipsec policy" ipsec-policy=in,ipsec
add action=drop chain=input comment="defconf: drop everything else not coming from LAN" in-interface-list=!LAN
add action=accept chain=forward comment="defconf: accept established,related,untracked" connection-state=established,related,untracked
add action=drop chain=forward comment="defconf: drop invalid" connection-state=invalid
add action=drop chain=forward comment="defconf: drop packets with bad src ipv6" src-address-list=bad_ipv6
add action=drop chain=forward comment="defconf: drop packets with bad dst ipv6" dst-address-list=bad_ipv6
add action=drop chain=forward comment="defconf: rfc4890 drop hop-limit=1" hop-limit=equal:1 protocol=icmpv6
add action=accept chain=forward comment="defconf: accept ICMPv6" protocol=icmpv6
add action=accept chain=forward comment="defconf: accept HIP" protocol=139
add action=accept chain=forward comment="defconf: accept IKE" dst-port=500,4500 protocol=udp
add action=accept chain=forward comment="defconf: accept ipsec AH" protocol=ipsec-ah
add action=accept chain=forward comment="defconf: accept ipsec ESP" protocol=ipsec-esp
add action=accept chain=forward comment="defconf: accept all that matches ipsec policy" ipsec-policy=in,ipsec
add action=drop chain=forward comment="defconf: drop everything else not coming from LAN" in-interface-list=!LAN
```

Then setup the dhcpv6 client
```
/ipv6 dhcp-client
add interface=ether1 pool-name=0 rapid-commit=no request=address,prefix use-peer-dns=no
```

Assign the bridge (LAN) interface an address from the (future) incoming prefix
```
/ipv6 address
add address=::1 from-pool=0 interface=bridge
```

Enable IPv6 and start listning to incoming Router Advertisements
```
/ipv6 settings
set accept-redirects=no accept-router-advertisements=yes max-neighbor-entries=8192
```

And finally start broadcast Router Advertisements to your network and assign addresses to your clients through [SLAAC](https://en.wikipedia.org/wiki/IPv6#Stateless_address_autoconfiguration_(SLAAC))
```
/ipv6 nd
set [ find default=yes ] advertise-dns=no hop-limit=64 interface=bridge managed-address-configuration=yes other-configuration=yes
```

Hope this will help all other (🤣) MikroTik users in Sundbybergs stadsnät. The only issue I experienced was that the route to my prefix was stuck against previous router (another address) so it took some time before upstream router sent the traffic the right way.

Must say that I'm very impressed by MikroTik so far. My first experience with the brand and everything is solid and seems priceworthy.
