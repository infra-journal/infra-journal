# WireGuard VPN Setup Guide

## Steps

1. Access the URL: [https://www.wireguard.com/install/](https://www.wireguard.com/install/) and click on **Download Windows Installer**.

2. Install the `.exe` file.

3. Run the WireGuard application and click **"Import tunnel(s) from file"**.

4. Select the client configuration file (`.conf`).

5. Click **Activate** to connect.

   Once the tunnel is imported, you will see the interface details:

   **Interface**
   | Field       | Value       |
   |-------------|-------------|
   | Status      | Inactive    |
   | Addresses   | 10.0.0.2/32 |
   | DNS Servers | 1.1.1.1     |

   **Peer**
   | Field                | Value |
   |----------------------|-------|
   | Allowed IPs          |       |
   | Endpoint             |       |
   | Persistent Keepalive | 25    |

   Click **Activate** to establish the VPN connection.
