# Enterprise Network Lab in Cisco Packet Tracer

This project models the network of Xpto, Lda across four sites in Portugal. It combines departmental VLANs, redundant Layer 2 switching in Lisbon, wireless access, and DHCP and HTTP services. Two Packet Tracer files let you compare static routing and RIP across the same inter-site ring design.

## Packet Tracer files

Keep both project files in the same directory as this README.

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

## Run and validate the lab

Open either `.pkt` file in Cisco Packet Tracer. Start in **Realtime** mode for connectivity checks, then switch to **Simulation** mode to inspect protocol exchanges. Run the same inter-site tests in both files to compare the routing behavior.

| Check | How to validate |
| --- | --- |
| Same-VLAN connectivity | Ping between two PCs in the same VLAN. Check port membership with `show vlan brief` on the switch. |
| Inter-VLAN connectivity | Ping each client's default gateway and then a PC in another VLAN. Check switch trunks with `show interfaces trunk` and router subinterfaces with `show ip interface brief`. |
| Lisbon Layer 2 redundancy | Use `show spanning-tree` to inspect the root bridge and port roles. Temporarily disconnect a forwarding link and test connectivity again after STP converges. |
| DHCP and HTTP | Set a client to DHCP and check its address, mask, and default gateway with `ipconfig`. Open the HTTP server's IPv4 address in the client's browser. |
| Inter-site routing | Ping and run `tracert` from a PC to a remote site. Compare `show ip route` on the routers in both files for static (`S`) and RIP-learned (`R`) routes to remote networks. |
| Protocol exchanges | In Simulation mode, filter for ARP and DHCP to observe address resolution and lease assignment. In the RIP file, also filter for RIP to inspect routing updates between routers. |

For an inter-site failure test, temporarily disconnect one router-to-router link in each file. After the routing state settles, compare the routing table and end-to-end reachability. In the static scenario, the result depends on the configured routes and any backup routes.
