# Fix: Remote Access Setting Not Persisting After Reboot

## Problem
The "Enable Remote Access" checkbox on the Gateway Config page doesn't persist after rebooting the gateway. The cloudflared tunnel stops on reboot even when it was enabled.

## Root Cause
There's a key name mismatch in the Node-RED flows. Some places use `cloudflare_tunnel_enabled` while others use `remote_access_enabled`.

---

## Step 1: Update Node-RED Functions

Open Node-RED admin at `http://<gateway-ip>:1880/admin`

Login with credentials: `ncdio / ncdB3ast`

### Node 1: "Format config response"

1. Find the node named **"Format config response"** (in the Gateway Config API section)
2. Double-click to edit
3. Find this line in the function:
   ```javascript
   if (config.cloudflare_tunnel_enabled !== undefined) {
       config.cloudflare_tunnel_enabled = config.cloudflare_tunnel_enabled === 'true' || config.cloudflare_tunnel_enabled === true;
   } else {
       config.cloudflare_tunnel_enabled = false;
   }
   ```
4. Replace it with:
   ```javascript
   if (config.remote_access_enabled !== undefined) {
       config.remote_access_enabled = config.remote_access_enabled === 'true' || config.remote_access_enabled === true;
   } else {
       config.remote_access_enabled = false;
   }
   ```
5. Click **Done**

### Node 2: "Update gateway config"

1. Find the node named **"Update gateway config"** (in the Gateway Config API section)
2. Double-click to edit
3. Find and replace ALL instances of `cloudflare_tunnel_enabled` with `remote_access_enabled`
   
   Specifically, change:
   ```javascript
   flow.set('cloudflare_tunnel_enabled', cfEnabled);
   ```
   to:
   ```javascript
   flow.set('remote_access_enabled', cfEnabled);
   ```
   
   And change:
   ```javascript
   { key: 'cloudflare_tunnel_enabled', value: String(cfEnabled) },
   ```
   to:
   ```javascript
   { key: 'remote_access_enabled', value: String(cfEnabled) },
   ```
4. Click **Done**

### Node 3: "Format Status Response"

1. Find the node named **"Format Status Response"** (in the Cloudflare Tunnel API section)
2. Double-click to edit
3. Find this line:
   ```javascript
   const enabled = flow.get('cloudflare_tunnel_enabled') || false;
   ```
4. Replace it with:
   ```javascript
   const enabled = flow.get('remote_access_enabled') || false;
   ```
5. Click **Done**

### Deploy Changes

Click the red **Deploy** button in the top-right corner of Node-RED.

---

## Step 2: Update the Database

SSH into the gateway:
```bash
ssh ncdio@<gateway-address>.local
```

Run this command to rename the existing database key:
```bash
sqlite3 /overlay/telemetry.db "UPDATE gateway_config SET key = 'remote_access_enabled' WHERE key = 'cloudflare_tunnel_enabled';"
```

Verify the change:
```bash
sqlite3 /overlay/telemetry.db "SELECT * FROM gateway_config WHERE key LIKE '%access%' OR key LIKE '%tunnel%';"
```

You should see `remote_access_enabled` in the output.

---

## Step 3: Test the Fix

1. Go to the Gateway Config page in the web UI
2. Check "Enable Remote Access" 
3. Click "Save Configuration"
4. Verify the tunnel starts: `pm2 status` should show cloudflared as "online"
5. **Reboot the gateway**: `sudo reboot`
6. Wait for gateway to come back online
7. SSH in and check: `pm2 status` - cloudflared should still be "online"
8. Verify in web UI - the "Enable Remote Access" checkbox should still be checked

---

## Summary of Changes

| Location | Change |
|----------|--------|
| Node-RED: "Format config response" | `cloudflare_tunnel_enabled` → `remote_access_enabled` |
| Node-RED: "Update gateway config" | `cloudflare_tunnel_enabled` → `remote_access_enabled` |
| Node-RED: "Format Status Response" | `cloudflare_tunnel_enabled` → `remote_access_enabled` |
| Database | Rename key from `cloudflare_tunnel_enabled` to `remote_access_enabled` |
