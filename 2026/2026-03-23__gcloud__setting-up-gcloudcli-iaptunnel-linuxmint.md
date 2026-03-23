# gcloud CLI Setup & IAP Tunnel Guide

## Step 1 — Install gcloud CLI

```bash
sudo apt-get install apt-transport-https ca-certificates gnupg curl

curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo gpg --dearmor -o /usr/share/keyrings/cloud.google.gpg

echo "deb [signed-by=/usr/share/keyrings/cloud.google.gpg] https://packages.cloud.google.com/apt cloud-sdk main" | sudo tee /etc/apt/sources.list.d/google-cloud-sdk.list

sudo apt-get update && sudo apt-get install google-cloud-cli
```

---

## Step 2 — Authenticate Once

```bash
gcloud auth login
```

Opens a browser — sign in with your Google account. You only do this **once** regardless of how many projects you have.

---

## Step 3 — Create a Named Config for Each Project

Create one config per project. Add as many projects as you like.

```bash
# Project 1
gcloud config configurations create proj1
gcloud config set account your@gmail.com
gcloud config set project YOUR_PROJECT_ID_1

# Project 2
gcloud config configurations create proj2
gcloud config set account your@gmail.com
gcloud config set project YOUR_PROJECT_ID_2

# ... repeat for each project
```

> **Tip:** Use meaningful aliases like `prod`, `staging`, `client-x` instead of `proj1`, `proj2` etc.

Verify all your configs anytime:

```bash
gcloud config configurations list
```

---

## Step 4 — Connecting to VMs

### Linux VMs (SSH)

```bash
gcloud config configurations activate proj1

gcloud compute ssh VM_NAME \
  --zone=ZONE \
  --tunnel-through-iap
```

### Windows VMs (RDP via IAP Tunnel)

IAP doesn't do RDP directly — you create a local tunnel first, then connect your RDP client to it.

**Terminal 1 — Open the tunnel and keep it running:**

```bash
gcloud config configurations activate proj1

gcloud compute start-iap-tunnel VM_NAME 3389 \
  --local-host-port=localhost:13389 \
  --zone=ZONE
```

**Then connect Remmina (or any RDP client) to:**

```
localhost:13389
```

Use the Windows VM's username and password as usual.

> Port 3389 is **never exposed publicly** — traffic routes through Google's IAP network using your authenticated account only.

---

## Step 5 — Aliases for Quick Access

Add these to `~/.bashrc` to avoid typing long commands every time:

```bash
# Switch project configs
alias gcp-proj1='gcloud config configurations activate proj1'
alias gcp-proj2='gcloud config configurations activate proj2'

# SSH shortcuts (Linux VMs)
alias ssh-vm1='gcloud config configurations activate proj1 && gcloud compute ssh VM_NAME --zone=ZONE --tunnel-through-iap'

# RDP tunnel shortcuts (Windows VMs)
alias rdp-vm1='gcloud config configurations activate proj1 && gcloud compute start-iap-tunnel WIN_VM_NAME 3389 --local-host-port=localhost:13389 --zone=ZONE'
```

Reload after editing:

```bash
source ~/.bashrc
```

---

## Quick Reference

| Task | Command |
|---|---|
| Switch project | `gcloud config configurations activate ALIAS` |
| SSH into Linux VM | `gcloud compute ssh VM_NAME --zone=ZONE --tunnel-through-iap` |
| Open RDP tunnel | `gcloud compute start-iap-tunnel VM_NAME 3389 --local-host-port=localhost:13389 --zone=ZONE` |
| List all configs | `gcloud config configurations list` |
| List VMs in active project | `gcloud compute instances list` |
| Re-authenticate | `gcloud auth login` |
