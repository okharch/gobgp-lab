# Docker Compose Network with DNS Resolution

This project sets up a simulated network environment using Docker Compose, consisting of two hosts, two routers, and a dedicated DNS server. 
The configuration demonstrates basic IP routing between different subnets and integrates DNS for name resolution, 
allowing hosts to communicate using names instead of just IP addresses.

## Network Topology

The network consists of three distinct bridge networks:

* **`net-router1`**: Connects `host1` to `router1`.
    * Subnet: `172.20.0.0/24`
    * `host1` IP: `172.20.0.100`
    * `router1` IP (on this net): `172.20.0.254`
    * `dns_server` IP (on this net): `172.20.0.253`

* **`net-router2`**: Connects `host2` to `router2`.
    * Subnet: `172.20.1.0/24`
    * `host2` IP: `172.20.1.100`
    * `router2` IP (on this net): `172.20.1.254`
    * `dns_server` IP (on this net): `172.20.1.253`

* **`net-core`**: Connects `router1` and `router2` to facilitate communication between the two `net-router` networks.
    * Subnet: `172.20.3.0/24`
    * `router1` IP (on this net): `172.20.3.101`
    * `router2` IP (on this net): `172.20.3.2`
    * `dns_server` IP (on this net): `172.20.3.253`

### Routing Configuration:

* **`router1`**: Routes traffic for `172.20.1.0/24` (host2's network) via `router2`'s `net-core` interface (`172.20.3.2`).
* **`router2`**: Routes traffic for `172.20.0.0/24` (host1's network) via `router1`'s `net-core` interface (`172.20.3.101`).
* **`host1` and `host2`**: Each host's default route points to its respective router.

### DNS Services:

A `dns_server` container running `dnsmasq` provides DNS resolution for all hosts and routers. It has static entries for all named entities in the network:

* `host1`: `172.20.0.100`
* `host2`: `172.20.1.100`
* `router1`: `172.20.0.254`
* `router2`: `172.20.1.254`
* `router1-core`: `172.20.3.101`
* `router2-core`: `172.20.3.2`

Each host and router is configured to use the `dns_server`'s IP address on its local network segment (e.g., `host1` uses `172.20.0.253`).

## Prerequisites

* Docker Desktop (for Windows/macOS) or Docker Engine (for Linux)
* Docker Compose (usually bundled with Docker Desktop, or installed separately)

## Setup and Usage

1.  **Save the Docker Compose file:**
    Save the provided Docker Compose configuration as `docker-compose.yaml` in your project directory.

2.  **Bring up the network:**
    ```bash
    docker compose up -d
    ```
    This command will download the `alpine` image (if not already present), create the networks, and start all containers.

3.  **Verify DNS resolution:**
    You can exec into any host and try to `ping` or `traceroute` other hosts by their names.
    For example, to access `host1`'s shell:
    ```bash
    docker exec -it host1 ash
    ```
    Inside the `host1` shell:
    * First, install `iputils` for `ping` and `traceroute`:
        ```bash
        apk add --no-cache iputils-ping iputils-traceroute
        ```
    * Then, try to ping `host2` by its name:
        ```bash
        ping host2
        ```
      You should see successful replies and `host2`'s IP address being resolved.

    * Now, try a `traceroute` to `host2` by its name:
        ```bash
        traceroute host2
        ```
      The output should show the names of the intermediate routers (`router1`, `router2-core`, `router2`) in addition to their IP addresses.

      Example `traceroute` output from `host1` to `host2`:

        ```
        traceroute to host2 (172.20.1.100), 30 hops max, 46 byte packets
         1  router1 (172.20.0.254)  0.370 ms  0.137 ms  0.105 ms
         2  router2-core (172.20.3.2)  0.428 ms  0.426 ms  0.395 ms
         3  router2 (172.20.1.254)  0.505 ms  0.540 ms  0.478 ms
         4  host2 (172.20.1.100)  0.391 ms  0.362 ms  0.362 ms
        ```

