# Device Configurations

Full configuration for every device in the lab.


Apply order: routers and IOU switches first, then the ASAs.

## ISP

```
enable
configure terminal
hostname ISP
interface f0/0
 ip address 11.3.0.1 255.255.255.252
 no shutdown
interface f1/0
 ip address 11.4.0.1 255.255.255.252
 no shutdown
interface f1/1
 ip address 11.5.0.1 255.255.255.252
 no shutdown
exit
ip route 11.0.0.0 255.255.255.252 11.3.0.2
ip route 11.1.0.0 255.255.255.252 11.4.0.2
ip route 11.2.0.0 255.255.255.252 11.5.0.2
end
write memory
```

## HQ

```
enable
configure terminal
hostname HQ
interface f0/0
 ip address 11.3.0.2 255.255.255.252
 no shutdown
interface f1/0
 ip address 11.0.0.1 255.255.255.252
 no shutdown
exit
ip route 0.0.0.0 0.0.0.0 11.3.0.1
end
write memory
```

## B1

```
enable
configure terminal
hostname B1
interface f0/0
 ip address 11.4.0.2 255.255.255.252
 no shutdown
interface f1/0
 ip address 11.1.0.1 255.255.255.252
 no shutdown
exit
ip route 0.0.0.0 0.0.0.0 11.4.0.1
end
write memory
```

## B2

```
enable
configure terminal
hostname B2
interface f0/0
 ip address 11.5.0.2 255.255.255.252
 no shutdown
interface f1/0
 ip address 11.2.0.1 255.255.255.252
 no shutdown
exit
ip route 0.0.0.0 0.0.0.0 11.5.0.1
end
write memory
```

## IOU1

```
enable
configure terminal
hostname IOU1
ip routing
vlan 10
 exit
vlan 20
 exit
interface e0/0
 no switchport
 ip address 192.168.70.2 255.255.255.0
 no shutdown
interface e0/1
 switchport mode access
 switchport access vlan 10
 no shutdown
interface e0/2
 switchport mode access
 switchport access vlan 20
 no shutdown
interface vlan 10
 ip address 192.168.10.1 255.255.255.0
 no shutdown
interface vlan 20
 ip address 192.168.20.1 255.255.255.0
 no shutdown
exit
ip dhcp excluded-address 192.168.10.1
ip dhcp excluded-address 192.168.20.1
ip dhcp pool VLAN10
 network 192.168.10.0 255.255.255.0
 default-router 192.168.10.1
 dns-server 8.8.8.8
ip dhcp pool VLAN20
 network 192.168.20.0 255.255.255.0
 default-router 192.168.20.1
 dns-server 8.8.8.8
exit
ip route 0.0.0.0 0.0.0.0 192.168.70.1
end
write memory
```

## IOU2

```
enable
configure terminal
hostname IOU2
ip routing
vlan 30
 exit
vlan 40
 exit
interface e0/0
 no switchport
 ip address 192.168.80.2 255.255.255.0
 no shutdown
interface e0/1
 switchport mode access
 switchport access vlan 30
 no shutdown
interface e0/2
 switchport mode access
 switchport access vlan 40
 no shutdown
interface vlan 30
 ip address 192.168.30.1 255.255.255.0
 no shutdown
interface vlan 40
 ip address 192.168.40.1 255.255.255.0
 no shutdown
exit
ip dhcp excluded-address 192.168.30.1
ip dhcp excluded-address 192.168.40.1
ip dhcp pool VLAN30
 network 192.168.30.0 255.255.255.0
 default-router 192.168.30.1
 dns-server 8.8.8.8
ip dhcp pool VLAN40
 network 192.168.40.0 255.255.255.0
 default-router 192.168.40.1
 dns-server 8.8.8.8
exit
ip route 0.0.0.0 0.0.0.0 192.168.80.1
end
write memory
```

## IOU3

```
enable
configure terminal
hostname IOU3
ip routing
vlan 50
 exit
vlan 60
 exit
interface e0/0
 no switchport
 ip address 192.168.90.2 255.255.255.0
 no shutdown
interface e0/1
 switchport mode access
 switchport access vlan 50
 no shutdown
interface e0/2
 switchport mode access
 switchport access vlan 60
 no shutdown
interface vlan 50
 ip address 192.168.50.1 255.255.255.0
 no shutdown
interface vlan 60
 ip address 192.168.60.1 255.255.255.0
 no shutdown
exit
ip dhcp excluded-address 192.168.50.1
ip dhcp excluded-address 192.168.60.1
ip dhcp pool VLAN50
 network 192.168.50.0 255.255.255.0
 default-router 192.168.50.1
 dns-server 8.8.8.8
ip dhcp pool VLAN60
 network 192.168.60.0 255.255.255.0
 default-router 192.168.60.1
 dns-server 8.8.8.8
exit
ip route 0.0.0.0 0.0.0.0 192.168.90.1
end
write memory
```

## ASA1 (HQ)

```
enable
configure terminal
hostname ASA1

interface GigabitEthernet0/0
 nameif INSIDE
 security-level 100
 ip address 192.168.70.1 255.255.255.0
 no shutdown
interface GigabitEthernet0/1
 nameif OUTSIDE
 security-level 0
 ip address 11.0.0.2 255.255.255.252
 no shutdown
interface Management0/0
 management-only
 nameif management
 security-level 100
 ip address 192.168.1.1 255.255.255.0
 no shutdown
exit

route OUTSIDE 0.0.0.0 0.0.0.0 11.0.0.1 1
route INSIDE 192.168.10.0 255.255.255.0 192.168.70.2 1
route INSIDE 192.168.20.0 255.255.255.0 192.168.70.2 1

object network HQ-VLAN10
 subnet 192.168.10.0 255.255.255.0
 nat (INSIDE,OUTSIDE) dynamic interface
object network HQ-VLAN20
 subnet 192.168.20.0 255.255.255.0
 nat (INSIDE,OUTSIDE) dynamic interface
object network HQ-MLS
 subnet 192.168.70.0 255.255.255.0
 nat (INSIDE,OUTSIDE) dynamic interface
object network B1-VLAN30
 subnet 192.168.30.0 255.255.255.0
object network B1-VLAN40
 subnet 192.168.40.0 255.255.255.0
object network B1-MLS
 subnet 192.168.80.0 255.255.255.0
object network B2-VLAN50
 subnet 192.168.50.0 255.255.255.0
object network B2-VLAN60
 subnet 192.168.60.0 255.255.255.0
object network B2-MLS
 subnet 192.168.90.0 255.255.255.0
exit

object-group network HQ_Office
 network-object object HQ-VLAN10
 network-object object HQ-VLAN20
 network-object object HQ-MLS
object-group network B1_Office
 network-object object B1-VLAN30
 network-object object B1-VLAN40
 network-object object B1-MLS
object-group network B2_Office
 network-object object B2-VLAN50
 network-object object B2-VLAN60
 network-object object B2-MLS
exit

access-list INSIDE_in extended permit ip any any
access-group INSIDE_in in interface INSIDE

policy-map global_policy
 class inspection_default
  inspect icmp
exit
exit

nat (INSIDE,OUTSIDE) source static HQ_Office HQ_Office destination static B1_Office B1_Office no-proxy-arp route-lookup
nat (INSIDE,OUTSIDE) source static HQ_Office HQ_Office destination static B2_Office B2_Office no-proxy-arp route-lookup

access-list VPN-TO-B1 extended permit ip object-group HQ_Office object-group B1_Office
access-list VPN-TO-B2 extended permit ip object-group HQ_Office object-group B2_Office

crypto ikev1 enable OUTSIDE
crypto ikev1 policy 1
 authentication pre-share
 encryption aes
 hash sha
 group 5
 lifetime 86400
exit
crypto ipsec ikev1 transform-set VPN-SET esp-aes esp-sha-hmac

tunnel-group 11.1.0.2 type ipsec-l2l
tunnel-group 11.1.0.2 ipsec-attributes
 ikev1 pre-shared-key CCNPSEC
exit
tunnel-group 11.2.0.2 type ipsec-l2l
tunnel-group 11.2.0.2 ipsec-attributes
 ikev1 pre-shared-key CCNPSEC
exit

crypto map OUTSIDE_map 10 match address VPN-TO-B1
crypto map OUTSIDE_map 10 set peer 11.1.0.2
crypto map OUTSIDE_map 10 set ikev1 transform-set VPN-SET
crypto map OUTSIDE_map 20 match address VPN-TO-B2
crypto map OUTSIDE_map 20 set peer 11.2.0.2
crypto map OUTSIDE_map 20 set ikev1 transform-set VPN-SET
crypto map OUTSIDE_map interface OUTSIDE

end
write memory
```

## ASA2 (B1)

```
enable
configure terminal
hostname ASA2

interface GigabitEthernet0/0
 nameif INSIDE
 security-level 100
 ip address 192.168.80.1 255.255.255.0
 no shutdown
interface GigabitEthernet0/1
 nameif OUTSIDE
 security-level 0
 ip address 11.1.0.2 255.255.255.252
 no shutdown
interface Management0/0
 management-only
 nameif management
 security-level 100
 ip address 192.168.1.2 255.255.255.0
 no shutdown
exit

route OUTSIDE 0.0.0.0 0.0.0.0 11.1.0.1 1
route INSIDE 192.168.30.0 255.255.255.0 192.168.80.2 1
route INSIDE 192.168.40.0 255.255.255.0 192.168.80.2 1

object network HQ-VLAN10
 subnet 192.168.10.0 255.255.255.0
object network HQ-VLAN20
 subnet 192.168.20.0 255.255.255.0
object network HQ-MLS
 subnet 192.168.70.0 255.255.255.0
object network B1-VLAN30
 subnet 192.168.30.0 255.255.255.0
 nat (INSIDE,OUTSIDE) dynamic interface
object network B1-VLAN40
 subnet 192.168.40.0 255.255.255.0
 nat (INSIDE,OUTSIDE) dynamic interface
object network B1-MLS
 subnet 192.168.80.0 255.255.255.0
 nat (INSIDE,OUTSIDE) dynamic interface
object network B2-VLAN50
 subnet 192.168.50.0 255.255.255.0
object network B2-VLAN60
 subnet 192.168.60.0 255.255.255.0
object network B2-MLS
 subnet 192.168.90.0 255.255.255.0
exit

object-group network HQ_Office
 network-object object HQ-VLAN10
 network-object object HQ-VLAN20
 network-object object HQ-MLS
object-group network B1_Office
 network-object object B1-VLAN30
 network-object object B1-VLAN40
 network-object object B1-MLS
object-group network B2_Office
 network-object object B2-VLAN50
 network-object object B2-VLAN60
 network-object object B2-MLS
exit

access-list INSIDE_in extended permit ip any any
access-group INSIDE_in in interface INSIDE

policy-map global_policy
 class inspection_default
  inspect icmp
exit
exit

nat (INSIDE,OUTSIDE) source static B1_Office B1_Office destination static HQ_Office HQ_Office no-proxy-arp route-lookup
nat (INSIDE,OUTSIDE) source static B1_Office B1_Office destination static B2_Office B2_Office no-proxy-arp route-lookup

access-list VPN-TO-HQ extended permit ip object-group B1_Office object-group HQ_Office
access-list VPN-TO-B2 extended permit ip object-group B1_Office object-group B2_Office

crypto ikev1 enable OUTSIDE
crypto ikev1 policy 1
 authentication pre-share
 encryption aes
 hash sha
 group 5
 lifetime 86400
exit
crypto ipsec ikev1 transform-set VPN-SET esp-aes esp-sha-hmac

tunnel-group 11.0.0.2 type ipsec-l2l
tunnel-group 11.0.0.2 ipsec-attributes
 ikev1 pre-shared-key CCNPSEC
exit
tunnel-group 11.2.0.2 type ipsec-l2l
tunnel-group 11.2.0.2 ipsec-attributes
 ikev1 pre-shared-key CCNPSEC
exit

crypto map OUTSIDE_map 10 match address VPN-TO-HQ
crypto map OUTSIDE_map 10 set peer 11.0.0.2
crypto map OUTSIDE_map 10 set ikev1 transform-set VPN-SET
crypto map OUTSIDE_map 20 match address VPN-TO-B2
crypto map OUTSIDE_map 20 set peer 11.2.0.2
crypto map OUTSIDE_map 20 set ikev1 transform-set VPN-SET
crypto map OUTSIDE_map interface OUTSIDE

end
write memory
```

## ASA3 (B2)

```
enable
configure terminal
hostname ASA3

interface GigabitEthernet0/0
 nameif INSIDE
 security-level 100
 ip address 192.168.90.1 255.255.255.0
 no shutdown
interface GigabitEthernet0/1
 nameif OUTSIDE
 security-level 0
 ip address 11.2.0.2 255.255.255.252
 no shutdown
interface Management0/0
 management-only
 nameif management
 security-level 100
 ip address 192.168.1.3 255.255.255.0
 no shutdown
exit

route OUTSIDE 0.0.0.0 0.0.0.0 11.2.0.1 1
route INSIDE 192.168.50.0 255.255.255.0 192.168.90.2 1
route INSIDE 192.168.60.0 255.255.255.0 192.168.90.2 1

object network HQ-VLAN10
 subnet 192.168.10.0 255.255.255.0
object network HQ-VLAN20
 subnet 192.168.20.0 255.255.255.0
object network HQ-MLS
 subnet 192.168.70.0 255.255.255.0
object network B1-VLAN30
 subnet 192.168.30.0 255.255.255.0
object network B1-VLAN40
 subnet 192.168.40.0 255.255.255.0
object network B1-MLS
 subnet 192.168.80.0 255.255.255.0
object network B2-VLAN50
 subnet 192.168.50.0 255.255.255.0
 nat (INSIDE,OUTSIDE) dynamic interface
object network B2-VLAN60
 subnet 192.168.60.0 255.255.255.0
 nat (INSIDE,OUTSIDE) dynamic interface
object network B2-MLS
 subnet 192.168.90.0 255.255.255.0
 nat (INSIDE,OUTSIDE) dynamic interface
exit

object-group network HQ_Office
 network-object object HQ-VLAN10
 network-object object HQ-VLAN20
 network-object object HQ-MLS
object-group network B1_Office
 network-object object B1-VLAN30
 network-object object B1-VLAN40
 network-object object B1-MLS
object-group network B2_Office
 network-object object B2-VLAN50
 network-object object B2-VLAN60
 network-object object B2-MLS
exit

access-list INSIDE_in extended permit ip any any
access-group INSIDE_in in interface INSIDE

policy-map global_policy
 class inspection_default
  inspect icmp
exit
exit

nat (INSIDE,OUTSIDE) source static B2_Office B2_Office destination static HQ_Office HQ_Office no-proxy-arp route-lookup
nat (INSIDE,OUTSIDE) source static B2_Office B2_Office destination static B1_Office B1_Office no-proxy-arp route-lookup

access-list VPN-TO-HQ extended permit ip object-group B2_Office object-group HQ_Office
access-list VPN-TO-B1 extended permit ip object-group B2_Office object-group B1_Office

crypto ikev1 enable OUTSIDE
crypto ikev1 policy 1
 authentication pre-share
 encryption aes
 hash sha
 group 5
 lifetime 86400
exit
crypto ipsec ikev1 transform-set VPN-SET esp-aes esp-sha-hmac

tunnel-group 11.0.0.2 type ipsec-l2l
tunnel-group 11.0.0.2 ipsec-attributes
 ikev1 pre-shared-key CCNPSEC
exit
tunnel-group 11.1.0.2 type ipsec-l2l
tunnel-group 11.1.0.2 ipsec-attributes
 ikev1 pre-shared-key CCNPSEC
exit

crypto map OUTSIDE_map 10 match address VPN-TO-HQ
crypto map OUTSIDE_map 10 set peer 11.0.0.2
crypto map OUTSIDE_map 10 set ikev1 transform-set VPN-SET
crypto map OUTSIDE_map 20 match address VPN-TO-B1
crypto map OUTSIDE_map 20 set peer 11.1.0.2
crypto map OUTSIDE_map 20 set ikev1 transform-set VPN-SET
crypto map OUTSIDE_map interface OUTSIDE

end
write memory
```

## PCs (VPCS)

On each PC:

```
ip dhcp
save
```
