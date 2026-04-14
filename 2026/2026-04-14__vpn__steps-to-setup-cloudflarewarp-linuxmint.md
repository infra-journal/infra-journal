# Cloudflare warp setup guide (on Linux Mint)

## 1. Add the Cloudflare Repository
```bash
# Add Cloudflare's GPG key
curl -fsSL https://pkg.cloudflareclient.com/pubkey.gpg | sudo gpg --yes --dearmor --output /usr/share/keyrings/cloudflare-warp-archive-keyring.gpg

# Add the repository
# Linux Mint 22 uses the codename zara, but Cloudflare's repo doesn't have a release for it. You need to use the Ubuntu base codename (noble) instead. 
echo "deb [signed-by=/usr/share/keyrings/cloudflare-warp-archive-keyring.gpg] https://pkg.cloudflareclient.com/ noble main" | sudo tee /etc/apt/sources.list.d/cloudflare-client.list
```

```bash
# DO NOT USE THIS 
echo "deb [signed-by=/usr/share/keyrings/cloudflare-warp-archive-keyring.gpg] https://pkg.cloudflareclient.com/ $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/cloudflare-client.list
```

## 2. Install WARP
```bash
sudo apt update && sudo apt install cloudflare-warp
```

## 3. Register and Connect
```bash
# Register a new client (first time only)
warp-cli registration new

# Connect
warp-cli connect
```

## 4. Verify It's Working
```bash
warp-cli status
```

> **You should see:**   
$warp-cli status  
Status update: Connected  
Network: healthy

---

## Common Commands
| Command | Description |
|---|---|
| warp-cli connect | Turn WARP on |
| warp-cli disconnect | Turn WARP off |
| warp-cli status | Check connection status |

