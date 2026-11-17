# ASA Site-to-Site IPsec VPN (HQ, B1, B2) on GNS3

Full-mesh site-to-site IPsec VPN between three Cisco ASAv firewalls, built in GNS3.
Based on the CCNP SCOR lab "Site to Site VPN using ASA Firewall" by Eng. Abeer Hosni.

## Implementation notes

- The original lab uses ASDM (GUI). This project implements the same configuration directly through the ASA CLI. The ASDM wizard only generates these same commands (visible in its "Preview CLI Commands" step).
- Changes compared to the original lab:
  - Fixed IOU2 `interface vlan 40`: the original reused `192.168.30.1`, corrected to `192.168.40.1`.
  - Extended the single tunnel (HQ to B1) into a full mesh: HQ-B1, HQ-B2, B1-B2.
- Target ASA version: 9.16 (matches ASDM 7.16 used in the original).

## Topology

Devices: 1 ISP router, 3 edge routers (HQ, B1, B2), 3 ASAv firewalls, 3 IOU L3 switches, 6 VPCS.

| # | Side A | Side B | Network |
|---|---|---|---|
| 1 | ISP f0/0 (.1) | HQ f0/0 (.2) | 11.3.0.0/30 |
| 2 | ISP f1/0 (.1) | B1 f0/0 (.2) | 11.4.0.0/30 |
| 3 | ISP f1/1 (.1) | B2 f0/0 (.2) | 11.5.0.0/30 |
| 4 | HQ f1/0 (.1) | ASA1 Gi0/1 (.2) | 11.0.0.0/30 |
| 5 | B1 f1/0 (.1) | ASA2 Gi0/1 (.2) | 11.1.0.0/30 |
| 6 | B2 f1/0 (.1) | ASA3 Gi0/1 (.2) | 11.2.0.0/30 |
| 7 | ASA1 Gi0/0 (.1) | IOU1 e0/0 (.2) | 192.168.70.0/24 |
| 8 | ASA2 Gi0/0 (.1) | IOU2 e0/0 (.2) | 192.168.80.0/24 |
| 9 | ASA3 Gi0/0 (.1) | IOU3 e0/0 (.2) | 192.168.90.0/24 |
| 10 | IOU1 e0/1 | PC1 | VLAN 10: 192.168.10.0/24 |
| 11 | IOU1 e0/2 | PC2 | VLAN 20: 192.168.20.0/24 |
| 12 | IOU2 e0/1 | PC3 | VLAN 30: 192.168.30.0/24 |
| 13 | IOU2 e0/2 | PC4 | VLAN 40: 192.168.40.0/24 |
| 14 | IOU3 e0/1 | PC5 | VLAN 50: 192.168.50.0/24 |
| 15 | IOU3 e0/2 | PC6 | VLAN 60: 192.168.60.0/24 |

Optional management network 192.168.1.0/24: ASA1 .1, ASA2 .2, ASA3 .3 (Management0/0 on a shared switch). Not required for the lab.

ASAv adapter mapping in GNS3: Adapter 0 = Management0/0, Adapter 1 = Gi0/0, Adapter 2 = Gi0/1.

Boot order: routers and IOU first, then the ASAv (3 to 5 minutes to boot).

## VPN policy

| Item | Value |
|---|---|
| IKE version | IKEv1 |
| Phase 1 | AES, SHA, pre-shared key, DH group 5, lifetime 86400 |
| Pre-shared key | CCNPSEC (lab key only) |
| Phase 2 transform set | VPN-SET, tunnel mode, esp-aes (128), esp-sha-hmac |
| NAT | Identity NAT (NAT exemption) for VPN traffic, PAT to interface for Internet |

## Configurations

### ISP

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

### HQ

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

### B1

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

### B2

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

### IOU1

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

### IOU2

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

### IOU3

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

### ASA1 (HQ)

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

### ASA2 (B1)

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

### ASA3 (B2)

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

### PCs (VPCS)

On each PC:

```
ip dhcp
save
```

## Verification

From PC1, ping a host in another site (e.g. VLAN 30 or VLAN 50). The first one or two pings may time out while the tunnel is being established. Also test Internet reachability with `ping 11.3.0.1`.

On the ASAs:

```
show crypto ikev1 sa
show crypto ipsec sa
show nat detail
```

Results observed on ASA1:

- Two IKEv1 SAs, one per peer (11.1.0.2 and 11.2.0.2), both in `MM_ACTIVE`.
- IPsec SAs active for each VLAN pair, transform `esp-aes esp-sha-hmac`, tunnel mode.
- `#pkts encaps/decaps` increasing, no send/recv errors.
- Manual (identity) NAT showing hits, so VPN traffic is exempt from PAT.

Proof that traffic is hidden: a Wireshark capture on the ISP to HQ link shows only ESP and ISAKMP between 11.0.0.2, 11.1.0.2 and 11.2.0.2. No ICMP is visible, and the ESP payload is unreadable.

## Troubleshooting notes

- Pasting long configs in one go can skip sections. The `object-group network` blocks were missed once, and the static NAT commands failed with `HQ_Office doesn't match an existing object or object-group`. Fix: create the object-groups, verify with `show running-config object-group`, then re-enter the NAT commands. Pasting in smaller batches avoids this.
- If a tunnel does not come up, first check peer reachability from the ASA (for example `ping 11.1.0.2` from ASA1). A failure there points to routing on the ISP or edge routers.

## Limitations (lab only)

- ASAv runs unlicensed in GNS3, so throughput is limited.
- IKEv1 with DH group 5 is used because the lab requires it. It is deprecated, and production should use IKEv2 with DH group 14 or higher and AES-256.
- `permit ip any any` on INSIDE is for lab simplicity only.
- `CCNPSEC` is a lab pre-shared key. Do not reuse it in production.

## Credits

Original lab: "Site to Site VPN using ASA Firewall for CCNP SCOR Students" by Eng. Abeer Hosni (2020).
