# 🕳️ Wirehole: Secure, Ad-Free, and Private Internet with WireGuard, Pi-hole, and Unbound

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Docker Hub](https://img.shields.io/badge/Docker-Images-blue)](https://hub.docker.com/r/pihole/pihole) <!-- Replace with actual image links if available -->

## ✨ Overview

Wirehole is a self-hosted, all-in-one solution designed to provide a secure, ad-free, and private internet experience for your devices, whether you're at home or on the go. By combining the power of **WireGuard VPN**, **Pi-hole** for network-wide ad blocking, and **Unbound** for a local recursive DNS resolver, Wirehole gives you full control over your DNS queries and protects your privacy by avoiding third-party DNS providers.

This setup is perfect for anyone looking to:
*   Block ads and trackers network-wide.
*   Encrypt their internet traffic through a personal VPN.
*   Improve privacy by resolving DNS queries directly, without relying on external DNS servers.
*   Access their home network securely from anywhere.

## 🚀 Features

*   **WireGuard VPN:** A fast, modern, and secure VPN protocol for encrypted remote access to your network.
*   **Pi-hole:** A DNS sinkhole that protects all your devices from unwanted content like ads, trackers, and malicious sites at the network level.
*   **Unbound (Recursive DNS):** Your very own validating, recursive, caching DNS resolver. This means your DNS queries go directly to the root servers, enhancing privacy and making you less reliant on your ISP's DNS.
*   **Docker-Compose Setup:** Easy deployment and management with Docker Compose, ensuring portability and isolated environments for each service.
*   **Customizable:** Easily configure environment variables and volumes to tailor the setup to your specific needs.

## 🏗️ Architecture

The Wirehole architecture leverages Docker containers to create a robust and interconnected system:

```
+------------------+     +-----------------+     +-----------------+
|   VPN Clients    |<--->|   WireGuard     |---->|     Pi-hole     |
| (Mobile/Laptop)  |     |  (wg-easy)      |<----+ (Ad-blocking,   |
+------------------+     |                 |     |   DNS Filtering)  |
                         +-----------------+     +-----------------+
                                                     |
                                                     V
                                               +-----------------+
                                               |     Unbound     |
                                               | (Recursive DNS) |
                                               +-----------------+
                                                     |
                                                     V
                                            +-----------------+
                                            |   Internet DNS  |
                                            |   (Root Servers)|
                                            +-----------------+
```

1.  **WireGuard (wg-easy):** Serves as the VPN endpoint. VPN clients connect here, encrypting their traffic.
2.  **Pi-hole:** All DNS queries from VPN clients (and optionally local network devices) are routed through Pi-hole, where ads and malicious domains are blocked.
3.  **Unbound:** Pi-hole forwards legitimate DNS queries to Unbound, which performs recursive lookups directly with the internet's root DNS servers, ensuring privacy and eliminating reliance on third-party DNS providers.

## 📋 Prerequisites

Before you begin, ensure you have the following installed on your server:

*   **Docker:** [Installation Guide](https://docs.docker.com/get-docker/)
*   **Docker Compose:** [Installation Guide](https://docs.docker.com/compose/install/)

## ⚙️ Installation and Setup

Follow these steps to get your Wirehole environment up and running:

1.  **Clone the Repository:**
    ```bash
    git clone https://github.com/your-username/wirehole.git # Replace with your repo URL
    cd wirehole
    ```

2.  **Create Environment Files:**
    Create a `.env` file in the root directory of the project. This file will store sensitive information and configuration variables.

    ```bash
    touch .env
    ```

    Open `.env` and add the following variables, replacing the placeholder values:

    ```env
    # Pi-hole Web UI Password
    FTLCONF_webserver_api_password='your_pihole_web_password'

    # WireGuard Public IP/Hostname (Your server's public IP address or domain)
    SERVERURL='your_server_public_ip_or_hostname'

    # WireGuard Web UI Password
    WGUI_PASSWORD='your_wireguard_web_password'

    # Timezone for Pi-hole container (e.g., Europe/Rome, America/New_York)
    TZ='Europe/Rome'
    ```
    **Note:** Make sure to choose strong, unique passwords for `FTLCONF_webserver_api_password` and `WGUI_PASSWORD`.

3.  **Adjust Unbound Configuration (Optional):**
    The Unbound configuration is located in the `unbound/` directory. You can customize `unbound.conf` and other related files if you have specific Unbound requirements. For most users, the default configuration should suffice.

4.  **Start the Services:**
    Navigate to the project root directory (where `docker-compose.yml` is located) and run:

    ```bash
    docker-compose up -d
    ```
    This command will download the necessary Docker images, create the containers, and start the services in detached mode.

5.  **Verify Services:**
    You can check the status of your running containers with:
    ```bash
    docker-compose ps
    ```
    All services (`unbound`, `pihole`, `wg-easy`) should be in the `Up` state.

## 🚀 Usage

### Accessing Pi-hole Admin Interface

After the containers are up, you can access the Pi-hole administration interface in your web browser:

*   **URL:** `http://your_server_public_ip_or_hostname/admin`
*   **Login:** Use the password you set for `FTLCONF_webserver_api_password` in your `.env` file.

From here, you can monitor DNS queries, view statistics, manage blacklists/whitelists, and configure Pi-hole.

### Accessing WireGuard Web UI (wg-easy)

Manage your WireGuard server and generate client configurations through the `wg-easy` web interface:

*   **URL:** `http://your_server_public_ip_or_hostname:51821`
*   **Login:** Use the password you set for `WGUI_PASSWORD` in your `.env` file.

From the `wg-easy` interface, you can:
*   Add new WireGuard clients.
*   Generate QR codes for easy client setup on mobile devices.
*   Download client configuration files.
*   Manage existing clients.

**Important:** When generating client configurations, ensure the DNS server is set to the Pi-hole IP address (`10.2.0.100`) as configured in `docker-compose.yml`.

### Connecting Clients to WireGuard

1.  **Install WireGuard Client:** Download and install the WireGuard client application on your desired device (desktop, mobile).
    *   [WireGuard Downloads](https://www.wireguard.com/install/)
2.  **Generate Client Configuration:** Use the `wg-easy` web UI to generate a new client.
3.  **Import Configuration:**
    *   **Mobile:** Scan the QR code using the WireGuard app.
    *   **Desktop:** Download the `.conf` file and import it into the WireGuard desktop application.
4.  **Activate VPN:** Connect to your Wirehole VPN from the WireGuard client application.

Once connected, all your internet traffic will be routed through your Wirehole server, leveraging Pi-hole for ad-blocking and Unbound for private DNS resolution.

## 🔧 Configuration

### `docker-compose.yml`

*   **Network Subnet:** The `private_network` defines the internal network for the containers. You can adjust the `subnet` if it conflicts with your existing network.
*   **Port Mappings:**
    *   `pihole`: `80:80/tcp` (Pi-hole Web UI), `53:53/tcp`, `53:53/udp` (DNS)
    *   `wg-easy`: `51820:51820/udp` (WireGuard VPN), `51821:51821/tcp` (WireGuard Web UI)
    Adjust these if they conflict with other services running on your host.
*   **Volume Mounts:**
    *   `./unbound:/opt/unbound/etc/unbound/`: Persists Unbound configurations.
    *   `./etc-pihole/:/etc/pihole/`: Persists Pi-hole configurations and data.
    *   `./etc-dnsmasq.d/:/etc/dnsmasq.d/`: For custom dnsmasq configurations for Pi-hole.
    *   `./wireguard-easy-data:/etc/wireguard`: Persists WireGuard configurations and client data.

### Environment Variables (`.env`)

*   `FTLCONF_webserver_api_password`: Pi-hole web interface password.
*   `SERVERURL`: Your server's public IP address or hostname. Essential for WireGuard client configurations.
*   `WGUI_PASSWORD`: WireGuard Web UI password.
*   `TZ`: Your timezone, e.g., `America/New_York`, `Europe/London`.

## 🤝 Contributing

We welcome contributions! If you have suggestions for improvements, new features, or bug fixes, please feel free to:

1.  Fork the repository.
2.  Create a new branch (`git checkout -b feature/your-feature`).
3.  Make your changes.
4.  Commit your changes (`git commit -m 'Add new feature'`).
5.  Push to the branch (`git push origin feature/your-feature`).
6.  Open a Pull Request.

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
