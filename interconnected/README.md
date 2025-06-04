# Docker Multi-Network Routing Lab

This project sets up a minimal Docker-based network topology to simulate a basic routing environment using Linux containers. It includes:

- Three isolated host containers (`host1`, `host2`, `host3`), each on its own subnet.
- One central router container (`router`) that connects all subnets and forwards packets.
- Static routing configured inside each host, using the router as their default gateway.
- IP forwarding enabled in the router (requires `privileged: true`).

## Network Topology

```
host1 (172.20.0.100) -- net-router1 --+
                                      |               
host2 (172.20.1.100) -- net-router2 --+--> router <-- net-core (172.20.3.254)
                                      |
host3 (172.20.2.100) -- net-router3 --+
```

## How It Works

- Each host container is assigned an IP in a separate `/24` subnet.
- The `router` container is connected to all three LAN networks (`net-router1`, `net-router2`, `net-router3`) and a backbone `net-core` network.
- Hosts have their default route set to the router's IP on their subnet.
- IP forwarding is enabled in the router container so it can route packets between networks.

## Usage

### 1. Start the environment

```bash
docker compose up -d
```

### 2. Verify routing

Run from your terminal:

```bash
docker exec host1 traceroute 172.20.1.100   # ping host2
docker exec host1 traceroute 172.20.2.100   # ping host3
docker exec host2 traceroute 172.20.0.100   # ping host1
docker exec host3 traceroute 172.20.1.100   # ping host2
```

You can also test the path using `traceroute` (install it via `apk add traceroute` if needed).

### 3. Clean up

```bash
docker compose down
```

## Notes

- The router uses `privileged: true` to modify kernel parameters like `/proc/sys/net/ipv4/ip_forward`.
- Hosts only require `NET_ADMIN` to modify their own routing tables.
- This setup does not use GoBGP yet — it provides a static routed foundation to build on.

## Next Steps

You can evolve this setup by:
- Adding GoBGP to the router container
- Advertising and learning routes dynamically
- Adding firewall rules (iptables)
- Simulating failures and failover scenarios

---

Created for routing experimentation using Docker and Alpine Linux.