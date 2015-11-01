---
layout: post
title:  "EdgeRouter IPv6 done right"
date:   2015-11-01 10:00:00
---
For little more than a month ago I bought an [Ubiquiti EdgeRouter PoE](https://www.ubnt.com/edgemax/edgerouter-poe/)
to replace my old Apple AirPort Extreme that couldn't handle the IPv6 [Prefix Deligation](https://en.wikipedia.org/wiki/Prefix_delegation) that my ISP provided.
The AirPort worked just fine when using the IPv6-in-IPv4 tunnel from [HE](https://tunnelbroker.net/) but wouldn't play when I got the native stuff.
From what I heard this could be the case on older models..

When I received my EdgeRouter I had never used one before. Never even owned any network device with an actual good CLI so this was new.
After some googling my understanding was that there where many ways to accomplish what I wanted, native IPv6!
Many of the alternative ways was to use SSH and modify files in the system like there was no tomorrow.
And from my understanding the community thougt this was good.

I oppose this!
None of the changes the user makes by modifing the system files will survive an upgrade or sometimes not even an reboot.
If you buy one of this advanced routers with a powerful CLI, use it!

I will not take any credit for the config I will post below. It's based on many posts from [Ubiqutis forum](https://community.ubnt.com) and random posts on the Internet.
But the config will hopefully make it through a system upgrade in the future and I hope it will help someone out there who tries to configure IPv6.

At least it's working almost perfect for me. Almost perfect. It's seems like I hit a [bug](https://community.ubnt.com/t5/EdgeMAX/The-generation-of-etc-radvd-conf-is-missing-my-configuration) regarding the DNS settings for the router advertisement.
But Ubiquti replied yesterday and said they fixed it in the latest alpha and from my testing it's now fixed.

So here is my config:

    firewall {
        all-ping enable
        broadcast-ping disable
        group {
            network-group BOGONS {
                network 10.0.0.0/8
                network 100.64.0.0/10
                network 127.0.0.0/8
                network 169.254.0.0/16
                network 172.16.0.0/12
                network 192.0.0.0/24
                network 192.0.2.0/24
                network 192.168.0.0/16
                network 198.18.0.0/15
                network 198.51.100.0/24
                network 203.0.113.0/24
                network 224.0.0.0/4
                network 240.0.0.0/4
            }
        }
        ipv6-name WANv6_IN {
            default-action drop
            description "WAN inbound traffic forwarded to LAN"
            enable-default-log
            rule 10 {
                action accept
                description "Allow established/related sessions"
                state {
                    established enable
                    related enable
                }
            }
            rule 20 {
                action drop
                description "Drop invalid state"
                state {
                    invalid enable
                }
            }
        }
        ipv6-name WANv6_LOCAL {
            default-action drop
            description "WAN inbound traffic to the router"
            enable-default-log
            rule 10 {
                action accept
                description "Allow established/related sessions"
                state {
                    established enable
                    related enable
                }
            }
            rule 20 {
                action drop
                description "Drop invalid state"
                state {
                    invalid enable
                }
            }
            rule 30 {
                action accept
                description "Allow IPv6 icmp"
                protocol ipv6-icmp
            }
            rule 40 {
                action accept
                description "allow dhcpv6"
                destination {
                    port 546
                }
                protocol udp
                source {
                    port 547
                }
            }
        }
        ipv6-receive-redirects disable
        ipv6-src-route disable
        ip-src-route disable
        log-martians enable
        name LAN_IN {
            default-action accept
            rule 1 {
                action drop
                description "drop invalid state"
                state {
                    invalid enable
                }
            }
        }
        name WAN_IN {
            default-action drop
            description "WAN to internal"
            rule 10 {
                action accept
                description "Allow established/related"
                state {
                    established enable
                    related enable
                }
            }
            rule 20 {
                action drop
                description "Drop invalid state"
                state {
                    invalid enable
                }
            }
        }
        name WAN_LOCAL {
            default-action drop
            description "WAN to router"
            rule 10 {
                action accept
                description "Allow established/related"
                state {
                    established enable
                    related enable
                }
            }
            rule 20 {
                action drop
                description "Drop invalid state"
                state {
                    invalid enable
                }
            }
        }
        receive-redirects disable
        send-redirects enable
        source-validation disable
        syn-cookies enable
    }
    interfaces {
        ethernet eth0 {
            address 192.168.1.1/24
            description "Local 2"
            duplex auto
            poe {
                output off
            }
            speed auto
        }
        ethernet eth1 {
            address dhcp
            description Internet
            dhcpv6-pd {
                no-dns
                pd 0 {
                    interface switch0 {
                        host-address ::1
                        no-dns
                        prefix-id :
                        service slaac
                    }
                    prefix-length /64
                }
                prefix-only
                rapid-commit disable
            }
            duplex auto
            firewall {
                in {
                    ipv6-name WANv6_IN
                    name WAN_IN
                }
                local {
                    ipv6-name WANv6_LOCAL
                    name WAN_LOCAL
                }
            }
            ipv6 {
                dup-addr-detect-transmits 1
            }
            poe {
                output off
            }
            speed auto
        }
        ethernet eth2 {
            description Local
            duplex auto
            poe {
                output off
            }
            speed auto
        }
        ethernet eth3 {
            description Local
            duplex auto
            poe {
                output off
            }
            speed auto
        }
        ethernet eth4 {
            description Local
            duplex auto
            poe {
                output 48v
            }
            speed auto
        }
        loopback lo {
        }
        switch switch0 {
            address 192.168.2.1/24
            description Local
            ipv6 {
                dup-addr-detect-transmits 1
                router-advert {
                    cur-hop-limit 64
                    link-mtu 0
                    managed-flag false
                    max-interval 600
                    name-server 2001:4860:4860::8888
                    name-server 2001:4860:4860::8844
                    other-config-flag false
                    prefix ::/64 {
                        autonomous-flag true
                        on-link-flag true
                        valid-lifetime 2592000
                    }
                    reachable-time 0
                    retrans-timer 0
                    send-advert true
                }
            }
            mtu 1500
            switch-port {
                interface eth2
                interface eth3
                interface eth4
            }
        }
    }
    service {
        dhcp-server {
            disabled false
            hostfile-update disable
            shared-network-name LAN1 {
                authoritative disable
                subnet 192.168.1.0/24 {
                    default-router 192.168.1.1
                    dns-server 8.8.8.8
                    dns-server 8.8.4.4
                    lease 86400
                    start 192.168.1.21 {
                        stop 192.168.1.240
                    }
                }
            }
            shared-network-name LAN2 {
                authoritative disable
                subnet 192.168.2.0/24 {
                    default-router 192.168.2.1
                    dns-server 8.8.4.4
                    dns-server 8.8.8.8
                    lease 86400
                    start 192.168.2.21 {
                        stop 192.168.2.240
                    }
                }
            }
        }
        gui {
            https-port 443
        }
        nat {
            rule 5010 {
                outbound-interface eth1
                type masquerade
            }
        }
        ssh {
            disable-password-authentication
            port 22
            protocol-version v2
        }
        ubnt-discover {
            disable
        }
    }
    system {
        host-name fooEdge
        login {
            user ubnt {
                authentication {
                    encrypted-password foo
                }
                level admin
            }
        }
        name-server 8.8.8.8
        name-server 2001:4860:4860::8888
        name-server 8.8.4.4
        name-server 2001:4860:4860::8844
        ntp {
            server 0.ubnt.pool.ntp.org {
            }
            server 1.ubnt.pool.ntp.org {
            }
            server 2.ubnt.pool.ntp.org {
            }
            server 3.ubnt.pool.ntp.org {
            }
        }
        syslog {
            global {
                facility all {
                    level notice
                }
                facility protocols {
                    level debug
                }
            }
        }
        time-zone Europe/Stockholm
        traffic-analysis {
            dpi disable
            export enable
        }
    }


or as pasteable commands:

    set firewall all-ping enable
    set firewall broadcast-ping disable
    set firewall group network-group BOGONS network 10.0.0.0/8
    set firewall group network-group BOGONS network 100.64.0.0/10
    set firewall group network-group BOGONS network 127.0.0.0/8
    set firewall group network-group BOGONS network 169.254.0.0/16
    set firewall group network-group BOGONS network 172.16.0.0/12
    set firewall group network-group BOGONS network 192.0.0.0/24
    set firewall group network-group BOGONS network 192.0.2.0/24
    set firewall group network-group BOGONS network 192.168.0.0/16
    set firewall group network-group BOGONS network 198.18.0.0/15
    set firewall group network-group BOGONS network 198.51.100.0/24
    set firewall group network-group BOGONS network 203.0.113.0/24
    set firewall group network-group BOGONS network 224.0.0.0/4
    set firewall group network-group BOGONS network 240.0.0.0/4
    set firewall ipv6-name WANv6_IN default-action drop
    set firewall ipv6-name WANv6_IN description 'WAN inbound traffic forwarded to LAN'
    set firewall ipv6-name WANv6_IN enable-default-log
    set firewall ipv6-name WANv6_IN rule 10 action accept
    set firewall ipv6-name WANv6_IN rule 10 description 'Allow established/related sessions'
    set firewall ipv6-name WANv6_IN rule 10 state established enable
    set firewall ipv6-name WANv6_IN rule 10 state related enable
    set firewall ipv6-name WANv6_IN rule 20 action drop
    set firewall ipv6-name WANv6_IN rule 20 description 'Drop invalid state'
    set firewall ipv6-name WANv6_IN rule 20 state invalid enable
    set firewall ipv6-name WANv6_LOCAL default-action drop
    set firewall ipv6-name WANv6_LOCAL description 'WAN inbound traffic to the router'
    set firewall ipv6-name WANv6_LOCAL enable-default-log
    set firewall ipv6-name WANv6_LOCAL rule 10 action accept
    set firewall ipv6-name WANv6_LOCAL rule 10 description 'Allow established/related sessions'
    set firewall ipv6-name WANv6_LOCAL rule 10 state established enable
    set firewall ipv6-name WANv6_LOCAL rule 10 state related enable
    set firewall ipv6-name WANv6_LOCAL rule 20 action drop
    set firewall ipv6-name WANv6_LOCAL rule 20 description 'Drop invalid state'
    set firewall ipv6-name WANv6_LOCAL rule 20 state invalid enable
    set firewall ipv6-name WANv6_LOCAL rule 30 action accept
    set firewall ipv6-name WANv6_LOCAL rule 30 description 'Allow IPv6 icmp'
    set firewall ipv6-name WANv6_LOCAL rule 30 protocol ipv6-icmp
    set firewall ipv6-name WANv6_LOCAL rule 40 action accept
    set firewall ipv6-name WANv6_LOCAL rule 40 description 'allow dhcpv6'
    set firewall ipv6-name WANv6_LOCAL rule 40 destination port 546
    set firewall ipv6-name WANv6_LOCAL rule 40 protocol udp
    set firewall ipv6-name WANv6_LOCAL rule 40 source port 547
    set firewall ipv6-receive-redirects disable
    set firewall ipv6-src-route disable
    set firewall ip-src-route disable
    set firewall log-martians enable
    set firewall name LAN_IN default-action accept
    set firewall name LAN_IN rule 1 action drop
    set firewall name LAN_IN rule 1 description 'drop invalid state'
    set firewall name LAN_IN rule 1 state invalid enable
    set firewall name WAN_IN default-action drop
    set firewall name WAN_IN description 'WAN to internal'
    set firewall name WAN_IN rule 10 action accept
    set firewall name WAN_IN rule 10 description 'Allow established/related'
    set firewall name WAN_IN rule 10 state established enable
    set firewall name WAN_IN rule 10 state related enable
    set firewall name WAN_IN rule 20 action drop
    set firewall name WAN_IN rule 20 description 'Drop invalid state'
    set firewall name WAN_IN rule 20 state invalid enable
    set firewall name WAN_LOCAL default-action drop
    set firewall name WAN_LOCAL description 'WAN to router'
    set firewall name WAN_LOCAL rule 10 action accept
    set firewall name WAN_LOCAL rule 10 description 'Allow established/related'
    set firewall name WAN_LOCAL rule 10 state established enable
    set firewall name WAN_LOCAL rule 10 state related enable
    set firewall name WAN_LOCAL rule 20 action drop
    set firewall name WAN_LOCAL rule 20 description 'Drop invalid state'
    set firewall name WAN_LOCAL rule 20 state invalid enable
    set firewall receive-redirects disable
    set firewall send-redirects enable
    set firewall source-validation disable
    set firewall syn-cookies enable
    set interfaces ethernet eth0 address 192.168.1.1/24
    set interfaces ethernet eth0 description 'Local 2'
    set interfaces ethernet eth0 duplex auto
    set interfaces ethernet eth0 poe output off
    set interfaces ethernet eth0 speed auto
    set interfaces ethernet eth1 address dhcp
    set interfaces ethernet eth1 description Internet
    set interfaces ethernet eth1 dhcpv6-pd no-dns
    set interfaces ethernet eth1 dhcpv6-pd pd 0 interface switch0 host-address '::1'
    set interfaces ethernet eth1 dhcpv6-pd pd 0 interface switch0 no-dns
    set interfaces ethernet eth1 dhcpv6-pd pd 0 interface switch0 prefix-id ':'
    set interfaces ethernet eth1 dhcpv6-pd pd 0 interface switch0 service slaac
    set interfaces ethernet eth1 dhcpv6-pd pd 0 prefix-length /64
    set interfaces ethernet eth1 dhcpv6-pd prefix-only
    set interfaces ethernet eth1 dhcpv6-pd rapid-commit disable
    set interfaces ethernet eth1 duplex auto
    set interfaces ethernet eth1 firewall in ipv6-name WANv6_IN
    set interfaces ethernet eth1 firewall in name WAN_IN
    set interfaces ethernet eth1 firewall local ipv6-name WANv6_LOCAL
    set interfaces ethernet eth1 firewall local name WAN_LOCAL
    set interfaces ethernet eth1 ipv6 dup-addr-detect-transmits 1
    set interfaces ethernet eth1 poe output off
    set interfaces ethernet eth1 speed auto
    set interfaces ethernet eth2 description Local
    set interfaces ethernet eth2 duplex auto
    set interfaces ethernet eth2 poe output off
    set interfaces ethernet eth2 speed auto
    set interfaces ethernet eth3 description Local
    set interfaces ethernet eth3 duplex auto
    set interfaces ethernet eth3 poe output off
    set interfaces ethernet eth3 speed auto
    set interfaces ethernet eth4 description Local
    set interfaces ethernet eth4 duplex auto
    set interfaces ethernet eth4 poe output 48v
    set interfaces ethernet eth4 speed auto
    set interfaces loopback lo
    set interfaces switch switch0 address 192.168.2.1/24
    set interfaces switch switch0 description Local
    set interfaces switch switch0 ipv6 dup-addr-detect-transmits 1
    set interfaces switch switch0 ipv6 router-advert cur-hop-limit 64
    set interfaces switch switch0 ipv6 router-advert link-mtu 0
    set interfaces switch switch0 ipv6 router-advert managed-flag false
    set interfaces switch switch0 ipv6 router-advert max-interval 600
    set interfaces switch switch0 ipv6 router-advert name-server '2001:4860:4860::8888'
    set interfaces switch switch0 ipv6 router-advert name-server '2001:4860:4860::8844'
    set interfaces switch switch0 ipv6 router-advert other-config-flag false
    set interfaces switch switch0 ipv6 router-advert prefix '::/64' autonomous-flag true
    set interfaces switch switch0 ipv6 router-advert prefix '::/64' on-link-flag true
    set interfaces switch switch0 ipv6 router-advert prefix '::/64' valid-lifetime 2592000
    set interfaces switch switch0 ipv6 router-advert reachable-time 0
    set interfaces switch switch0 ipv6 router-advert retrans-timer 0
    set interfaces switch switch0 ipv6 router-advert send-advert true
    set interfaces switch switch0 mtu 1500
    set interfaces switch switch0 switch-port interface eth2
    set interfaces switch switch0 switch-port interface eth3
    set interfaces switch switch0 switch-port interface eth4
    set service dhcp-server disabled false
    set service dhcp-server hostfile-update disable
    set service dhcp-server shared-network-name LAN1 authoritative disable
    set service dhcp-server shared-network-name LAN1 subnet 192.168.1.0/24 default-router 192.168.1.1
    set service dhcp-server shared-network-name LAN1 subnet 192.168.1.0/24 dns-server 8.8.8.8
    set service dhcp-server shared-network-name LAN1 subnet 192.168.1.0/24 dns-server 8.8.4.4
    set service dhcp-server shared-network-name LAN1 subnet 192.168.1.0/24 lease 86400
    set service dhcp-server shared-network-name LAN1 subnet 192.168.1.0/24 start 192.168.1.21 stop 192.168.1.240
    set service dhcp-server shared-network-name LAN2 authoritative disable
    set service dhcp-server shared-network-name LAN2 subnet 192.168.2.0/24 default-router 192.168.2.1
    set service dhcp-server shared-network-name LAN2 subnet 192.168.2.0/24 dns-server 8.8.4.4
    set service dhcp-server shared-network-name LAN2 subnet 192.168.2.0/24 dns-server 8.8.8.8
    set service dhcp-server shared-network-name LAN2 subnet 192.168.2.0/24 lease 86400
    set service dhcp-server shared-network-name LAN2 subnet 192.168.2.0/24 start 192.168.2.21 stop 192.168.2.240
    set service gui https-port 443
    set service nat rule 5010 outbound-interface eth1
    set service nat rule 5010 type masquerade
    set service ssh disable-password-authentication
    set service ssh port 22
    set service ssh protocol-version v2
    set service ubnt-discover disable
    set system login user ubnt authentication encrypted-password 'foo
    set system login user ubnt level admin
    set system name-server 8.8.8.8
    set system name-server '2001:4860:4860::8888'
    set system name-server 8.8.4.4
    set system name-server '2001:4860:4860::8844'
    set system ntp server 0.ubnt.pool.ntp.org
    set system ntp server 1.ubnt.pool.ntp.org
    set system ntp server 2.ubnt.pool.ntp.org
    set system ntp server 3.ubnt.pool.ntp.org
    set system syslog global facility all level notice
    set system syslog global facility protocols level debug
    set system time-zone Europe/Stockholm
    set system traffic-analysis dpi disable
    set system traffic-analysis export enable
