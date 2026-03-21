# GCP IAP Tunnel Setup — RDP Access to VM (Port 3389)

> **What is IAP Tunnel?**  
> Identity-Aware Proxy (IAP) TCP forwarding lets you access VM instances on GCP **without a public IP address or VPN**. All traffic is authenticated via your Google account and routed securely through Google's infrastructure.

---

## Architecture Overview

```
Your PC (localhost:13389)  →  Cloud IAP (Auth + Policy)  →  GCP VM (Port 3389)
                                        ↑
                              your@gmail.com (IAM role)
```

- No public IP required on the VM
- No VPN required
- Traffic encrypted end-to-end via HTTPS

---

## Prerequisites

- GCP project with a running VM (Windows)
- GCP Cloud Console access: [console.cloud.google.com](https://console.cloud.google.com)
- Your Gmail address (e.g. `your@gmail.com`)
- `gcloud` CLI installed (only needed for the local tunnel command)

---

## Method 1 — Cloud Console (GUI)

### Step 1 — Enable the IAP API

1. Go to [console.cloud.google.com](https://console.cloud.google.com)
2. In the top search bar, type **Identity-Aware Proxy** and click it
3. Click **Enable API** if prompted

---

### Step 2 — Add Your Gmail to IAM

1. Go to **IAM & Admin → IAM** in the left menu
2. Click **+ Grant Access** at the top
3. In the **New principals** field, enter your Gmail: `your@gmail.com`
4. Assign both roles:
   - `Cloud IAP` → **IAP-secured Tunnel User**
   - `Compute Engine` → **Compute Viewer**
5. Click **Save**

> Wait ~1 minute for the roles to propagate before proceeding.

---

### Step 3 — Create the Firewall Rule

IAP routes traffic from the IP range `35.235.240.0/20`. You must allow this range to reach port 3389 on your VM.

1. Go to **VPC Network → Firewall**
2. Click **+ Create Firewall Rule**
3. Fill in the fields as follows:

| Field | Value |
|---|---|
| Name | `allow-rdp-iap` |
| Network | `default` (or your VPC name) |
| Direction of traffic | Ingress |
| Action on match | Allow |
| Targets | Specified target tags |
| Target tags | `allow-rdp-iap` |
| Source filter | IPv4 ranges |
| Source IP ranges | `35.235.240.0/20` |
| Protocols and ports | TCP → `3389` |

4. Click **Create**

---

### Step 4 — Add the Network Tag to Your VM

1. Go to **Compute Engine → VM Instances**
2. Click your VM name → click **Edit** (pencil icon at the top)
3. Scroll down to **Network tags**
4. Add the tag: `allow-rdp-iap`
5. Click **Save**

---

### Step 5 — Connect via RDP

#### Using your own RDP client (e.g. Windows Remote Desktop)

Run this one command in your terminal to open the local tunnel:

```bash
gcloud compute start-iap-tunnel YOUR_VM_NAME 3389 \
  --local-host-port=localhost:13389 \
  --zone=YOUR_ZONE \
  --project=YOUR_PROJECT_ID
```

Keep the terminal open, then connect your RDP client to:

```
localhost:13389
```

---

## Option 2 — IAP Desktop App (Recommended for Frequent Use)

**IAP Desktop** is a free, open-source Windows application built by Google that provides a graphical interface for managing IAP tunnel connections. It is the easiest way to connect to GCP VMs regularly without running any commands.

### Download

- Official page: [github.com/GoogleCloudPlatform/iap-desktop](https://github.com/GoogleCloudPlatform/iap-desktop/releases)
- Download the latest `.msi` installer from the Releases page

### Features

- Visual list of all your GCP projects and VM instances
- One-click RDP connection through IAP tunnel
- Manages tunnel lifecycle automatically (no manual `gcloud` command needed)
- Supports multiple projects and instances simultaneously
- Runs SSH tunnels as well (port 22)
- Integrated Google Sign-In — uses the same Gmail you added to IAM

### How to Use

1. Install and launch **IAP Desktop**
2. Sign in with your Google account (`your@gmail.com`)
3. Your GCP projects and VM instances will be listed automatically
4. Right-click any Windows VM → click **Connect as administrator** or **Connect**
5. IAP Desktop opens the tunnel and launches RDP automatically

> No firewall rules or tunnel commands needed beyond what is already configured above. IAP Desktop handles everything in the background.

---

## Troubleshooting

| Issue | Fix |
|---|---|
| `ERROR: Permission denied` | Confirm `iap.tunnelResourceAccessor` role is assigned and ~1 min has passed |
| `Port already in use` | Change `13389` to another local port like `15389` |
| RDP connection refused after tunnel opens | Check the GCP firewall rule has the correct tag and the VM has that tag assigned |
| Tunnel connects but RDP fails | Ensure RDP service is enabled on the VM: `services.msc` → Remote Desktop Services → Running |
| IAP Desktop shows no instances | Ensure `Compute Viewer` role is assigned to your Gmail |

---

## Quick Reference — IAM Roles Required

| Role | Purpose |
|---|---|
| `roles/iap.tunnelResourceAccessor` | Allows tunneling through IAP |
| `roles/compute.viewer` | Allows listing and resolving VM instances |

## Quick Reference — Firewall Rule

| Setting | Value |
|---|---|
| Source IP range | `35.235.240.0/20` (GCP IAP range) |
| Protocol | TCP |
| Port | `3389` |
| Target | VMs tagged with `allow-rdp-iap` |

---

*Last updated: March 2026*
