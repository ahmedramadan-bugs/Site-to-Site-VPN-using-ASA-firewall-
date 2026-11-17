# Site-to-Site IPsec VPN on Cisco ASA (GNS3)

I built a full-mesh IPsec VPN between three Cisco ASAv firewalls in GNS3: HQ, B1 and B2. Any site can talk to any other site through an encrypted tunnel, and the ISP in the middle only ever sees ESP packets.

The lab comes from Eng. Abeer Hosni's CCNP SCOR material. It covers one tunnel between two sites, so I took it further.

<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/caab9244-eca0-4c41-aee9-8939b0295900" />

## What I did

- Built the three-site network: an edge router, an ASAv, an L3 switch and two VLANs per site, all connected through one ISP router.
- Applied everything from the CLI instead of ASDM (more on that below).
- Went from one tunnel to a full mesh (HQ-B1, HQ-B2, B1-B2).
- Set up PAT for Internet access, with NAT exemption so VPN traffic isn't translated.
- Checked it end to end with `show` commands, ping tests and a Wireshark capture.

## Why CLI and not ASDM?

The original lab uses ASDM, the ASA's GUI. It needs an extra management machine with Java on top of three ASAv already running in GNS3, and my machine couldn't handle all of that comfortably. So I did everything from the command line.

It's the same thing underneath. Every ASDM wizard ends with a "Preview CLI Commands" screen that shows the exact commands it sends to the firewall. The CLI just skips the clicking, and you end up with plain text you can copy, check and reuse.

## The VPN settings

| Phase | Setting | Value |
|---|---|---|
| 1 (IKEv1) | Encryption | AES |
| | Hash | SHA |
| | Authentication | Pre-shared key |
| | Key exchange | DH group 5 |
| | Lifetime | 86400 seconds |
| 2 (IPsec) | Transform set | `VPN-SET` |
| | Mode | Tunnel |
| | ESP encryption | AES-128 |
| | ESP authentication | SHA |

Sites and subnets:

| Site | Firewall public IP | Inside networks |
|---|---|---|
| HQ | 11.0.0.2 | 192.168.10.0/24, 192.168.20.0/24, 192.168.70.0/24 |
| B1 | 11.1.0.2 | 192.168.30.0/24, 192.168.40.0/24, 192.168.80.0/24 |
| B2 | 11.2.0.2 | 192.168.50.0/24, 192.168.60.0/24, 192.168.90.0/24 |

The full configs are in [CONFIGURATIONS.md](CONFIGURATIONS.md). Start the routers and switches first, then the ASAs.

## Does it work?

**All three firewalls have two active IKE tunnels, one per peer.**

<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/69a36ff4-8569-4d5c-bac1-0744efd23430" />



**PC1 reaches every other VLAN.** The first few pings time out while the tunnel comes up, then everything replies.

<img width="803" height="779" alt="image" src="https://github.com/user-attachments/assets/7600aac6-f58a-4f21-88ff-12adbdb8ac9b" />



**The encrypted packet counters go up with traffic, with no errors.**

<img width="990" height="668" alt="image" src="https://github.com/user-attachments/assets/1732063a-df61-4a47-abc8-67dcaf2485cf" />



**The ISP only sees ESP.** A capture on the ISP to HQ link shows no ICMP at all, just encrypted packets between the firewalls.

<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/b384aabb-d7f2-4814-9313-8a5b5f5dcab8" />


**The internal networks stay hidden.** The ISP's routing table only has the public 11.x.x.x networks, and Internet access from the PCs still works through NAT.

<img width="951" height="650" alt="image" src="https://github.com/user-attachments/assets/3d84dce5-0564-4db7-bfc0-822e700fc799" />

<img width="1139" height="489" alt="image" src="https://github.com/user-attachments/assets/cd21dac6-ac2b-4b32-a00b-15e73c8d1bc8" />


## Good to know

This is a lab. IKEv1 with DH group 5 is outdated, and a real deployment should use IKEv2 with a stronger group and AES-256. The ASAv is also unlicensed, so it's slow.

## Credits

Original lab by Eng. Abeer Hosni (CCNP SCOR, 2020). explained in these videos:

- https://www.youtube.com/watch?v=l507qFmetUU
- https://www.youtube.com/watch?v=fNNghq9IfgU
