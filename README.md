# Enterprise Network Lab in Cisco Packet Tracer

This project models the network of Xpto, Lda across four sites in Portugal. It combines departmental VLANs, redundant Layer 2 switching in Lisbon, wireless access, and DHCP and HTTP services. The project includes two implementations of the same inter-site ring, one using static routes and one using RIP.

## Packet Tracer files

| File | Inter-site routing |
| --- | --- |
| [StaticRoutes.pkt](./StaticRoutes.pkt) | Static routes |
| [RIPRouting.pkt](./RIPRouting.pkt) | Dynamic routing with RIP |

## Network design

The table lists the target capacity for each site.

| Site | Technical workstations | HR workstations | Additional networks |
| --- | ---: | ---: | --- |
| Lisbon | 40 (`DTEC-LX`) | 20 (`RH-LX`) | Wi-Fi for up to 100 concurrent devices and a server VLAN (`SRV`) sized for 10 servers, including DHCP and HTTP services |
| Porto | 20 | 10 | |
| Faro | 20 | 10 | |
| Ponta Delgada | 20 | 10 | |

Each site has a router and local switches. The four routers form a ring of point-to-point links.

```mermaid
flowchart LR
    LX["Lisbon"] --- PT["Porto"]
    PT --- PD["Ponta Delgada"]
    PD --- FA["Faro"]
    FA --- LX
```

- **VLANs and inter-VLAN routing.** Each department has its own VLAN. Trunk links carry multiple VLANs, and router subinterfaces provide the default gateways and enable traffic between VLANs.
- **Layer 2 redundancy.** Lisbon has redundant switch links. Spanning Tree Protocol (STP) keeps the active paths loop-free and can bring a backup path into service after a link failure.
- **Wireless access.** A dedicated switch connects the access points to the Lisbon Wi-Fi network.
- **Network services.** DHCP assigns IPv4 settings to clients, and the HTTP service provides an application-level connectivity check.

### IPv4 addressing

| Network | Address space |
| --- | --- |
| End devices and VLAN gateways | Subnets from `10.10.16.0/22` |
| Point-to-point links between routers | Individual `/30` subnets in `192.168.1.0/24` |

## Lab validation

Packet Tracer's **Realtime** mode supports end-to-end connectivity checks. **Simulation** mode shows packet exchanges such as ARP, DHCP, and RIP updates. The same tests apply to both routing implementations, with RIP updates specific to the dynamic scenario.

| Check | What to observe |
| --- | --- |
| Same-VLAN connectivity | A ping between two PCs in the same VLAN checks local reachability. The switch command `show vlan brief` shows access port membership. |
| Inter-VLAN connectivity | Pings to the default gateway and between VLANs check Layer 3 connectivity. The commands `show interfaces trunk` and `show ip interface brief` show the switch trunks and router subinterfaces. |
| Lisbon Layer 2 redundancy | The command `show spanning-tree` identifies the root bridge and port roles. A forwarding-link failure tests whether connectivity recovers after STP convergence. |
| DHCP and HTTP | The client command `ipconfig` shows the address, mask, and gateway assigned through DHCP. The HTTP server's IPv4 address provides a browser-based connectivity check. |
| Inter-site routing | The commands `ping` and `tracert` show remote reachability and the path taken. The router command `show ip route` displays static (`S`) or RIP-learned (`R`) routes to remote networks. |
| Protocol exchanges | Simulation mode displays ARP address resolution, DHCP lease assignment, and routing updates in the RIP scenario. |

An inter-router link failure provides a way to compare routing table changes and end-to-end reachability across both versions. In the static scenario, recovery depends on the configured routes and any backup routes.
