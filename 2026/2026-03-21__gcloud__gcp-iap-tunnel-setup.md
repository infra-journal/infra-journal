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

# GCP IAP Tunnel Setup — SSH Access to VM (Port 22)

> **Note:** This guide assumes you have already completed the base IAP setup for RDP (IAM roles assigned, IAP API enabled). If not, complete that first.

---

## What Changes for SSH

The IAM roles and IAP API are already in place from the RDP setup. You only need to:

1. Create a new firewall rule for port 22
2. Add a network tag to the VM
3. Use **IAP Desktop** to connect

---

## Step 1 — Create the Firewall Rule for SSH

1. Go to **VPC Network → Firewall**
2. Click **+ Create Firewall Rule**
3. Fill in the fields as follows:

| Field | Value |
|---|---|
| Name | `allow-ssh-iap` |
| Network | `default` (or your VPC name) |
| Direction of traffic | Ingress |
| Action on match | Allow |
| Targets | Specified target tags |
| Target tags | `allow-ssh-iap` |
| Source filter | IPv4 ranges |
| Source IP ranges | `35.235.240.0/20` |
| Protocols and ports | TCP → `22` |

4. Click **Create**

---

## Step 2 — Add the Network Tag to Your VM

1. Go to **Compute Engine → VM Instances**
2. Click your VM name → click **Edit** (pencil icon at the top)
3. Scroll down to **Network tags**
4. Add the tag: `allow-ssh-iap`
5. Click **Save**

> If the VM already has the `allow-rdp-iap` tag from the RDP setup, just add `allow-ssh-iap` alongside it — both tags can coexist on the same VM.

---

## Step 3 — Connect via SSH using IAP Desktop

**IAP Desktop** is the recommended way to connect via SSH through IAP. It handles the tunnel automatically with no commands needed.

### Download

- Official page: [github.com/GoogleCloudPlatform/iap-desktop](https://github.com/GoogleCloudPlatform/iap-desktop/releases)
- Download the latest `.msi` installer from the Releases page

### Connect

1. Install and launch **IAP Desktop**
2. Sign in with your Google account (`your@gmail.com`)
3. Your GCP projects and VM instances will appear automatically
4. Right-click your Linux VM → click **Connect**
5. IAP Desktop opens the IAP tunnel and SSH session automatically

> IAP Desktop picks a free local port on its own and manages the entire tunnel lifecycle — no terminal commands or manual port management required.

---

## Firewall Rules Summary

Both RDP and SSH use the same IAP source IP range. The only difference is the destination port and tag name.

| Protocol | VM Port | IAP Source Range | Firewall Tag |
|---|---|---|---|
| RDP | `3389` | `35.235.240.0/20` | `allow-rdp-iap` |
| SSH | `22` | `35.235.240.0/20` | `allow-ssh-iap` |

---

## Troubleshooting

| Issue | Fix |
|---|---|
| IAP Desktop shows no instances | Ensure `Compute Viewer` role is assigned to your Gmail in IAM |
| SSH connection refused | Check the firewall rule has the correct tag and the VM has `allow-ssh-iap` tag assigned |
| Permission denied (publickey) | Ensure your SSH key is added to the VM's `~/.ssh/authorized_keys` or use OS Login |
| IAP Desktop can't authenticate | Re-sign in via **File → Sign in** with the Gmail that has IAP roles assigned |

---

*Last updated: March 2026*
