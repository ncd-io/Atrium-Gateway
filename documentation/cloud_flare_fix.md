# Cloudflared Tunnel Fix Instructions

**Gateway:** ncd-a754.local  
**Issue:** Cloudflared PM2 service fails to start because config.yml is missing

## Problem Summary

The cloudflared service cannot start because the tunnel credentials and configuration file were never saved to the gateway. This typically happens when the tunnel was created from a different machine or the setup process was interrupted.

## Prerequisites

- SSH access to the gateway
- Username: `ncdio`
- Password (if needed for sudo): `ncdB3ast`

---

## Step-by-Step Fix

### Step 1: SSH into the Gateway

```bash
ssh ncdio@ncd-a754.local
```

### Step 2: Check Current State

Verify the problem exists:

```bash
ls -la ~/.cloudflared/
pm2 logs cloudflared --lines 10 --nostream
```

You should see `cert.pem` but NO `config.yml` and NO `.json` credentials file.

### Step 3: Check if Tunnel Exists

```bash
cloudflared tunnel list | grep a754
```

If a tunnel named `gateway-a754` exists but has no local credentials, you'll need to delete and recreate it.

### Step 4: Delete the Orphaned Tunnel (if exists)

```bash
cloudflared tunnel delete --force gateway-a754
```

### Step 5: Create New Tunnel

```bash
cloudflared tunnel create gateway-a754
```

**Important:** Note the tunnel ID from the output. It will look like:
```
Tunnel credentials written to /home/ncdio/.cloudflared/XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX.json
Created tunnel gateway-a754 with id XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX
```

### Step 6: Update DNS Route

```bash
cloudflared tunnel route dns --overwrite-dns gateway-a754 a754.iolight.com
```

### Step 7: Create Configuration File

Replace `TUNNEL_ID` with the actual tunnel ID from Step 5:

```bash
TUNNEL_ID="XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX"

cat > ~/.cloudflared/config.yml << EOF
tunnel: $TUNNEL_ID
credentials-file: /home/ncdio/.cloudflared/$TUNNEL_ID.json

ingress:
  - hostname: a754.iolight.com
    service: http://127.0.0.1:80
    originRequest:
      httpHostHeader: localhost
      connectTimeout: 10s
  - service: http_status:404
EOF
```

### Step 8: Verify Configuration

```bash
cat ~/.cloudflared/config.yml
cloudflared tunnel --config ~/.cloudflared/config.yml ingress validate
```

Should output: `OK`

### Step 9: Restart Cloudflared Service

```bash
pm2 restart cloudflared
pm2 save
```

### Step 10: Verify Service is Running

```bash
pm2 status
pm2 logs cloudflared --lines 15 --nostream
```

Look for lines like:
```
INF Registered tunnel connection connIndex=0 ... location=XXX protocol=quic
```

This confirms the tunnel is connected to Cloudflare's edge network.

---

## Verification

After completing these steps:

1. The gateway should be accessible at: `https://a754.iolight.com/`
2. PM2 status should show cloudflared as "online"
3. Logs should show multiple "Registered tunnel connection" messages

---

## Quick Reference Commands

| Action | Command |
|--------|---------|
| Check PM2 status | `pm2 status` |
| View cloudflared logs | `pm2 logs cloudflared --lines 20 --nostream` |
| Restart cloudflared | `pm2 restart cloudflared` |
| Stop cloudflared | `pm2 stop cloudflared` |
| Start cloudflared | `pm2 start cloudflared` |
| List tunnels | `cloudflared tunnel list` |
| Tunnel info | `cloudflared tunnel info gateway-a754` |

---

## If Issues Persist

1. Verify cert.pem exists: `ls -la ~/.cloudflared/cert.pem`
2. Verify credentials file exists: `ls -la ~/.cloudflared/*.json`
3. Verify config.yml exists: `cat ~/.cloudflared/config.yml`
4. Check for errors: `pm2 logs cloudflared --err --lines 50 --nostream`

If cert.pem is missing, you'll need to re-authenticate with Cloudflare:
```bash
cloudflared tunnel login
```
Then follow the URL to authenticate in a browser.
