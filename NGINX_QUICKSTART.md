# Quick Setup: Nginx DoH → DoH Ad Blocker

## 🎯 Your Setup

```
Browser/App
    ↓ HTTPS DoH
Nginx (your DoH upstream)
    ↓ logs queries
DoH Ad Blocker (learns & blocks)
```

## ⚡ Fast Track Setup

### Step 1: Configure Nginx Logging (2 minutes)

Choose your preferred nginx log format:

#### Option A: Standard Access Log with DNS Variables

Edit `/etc/nginx/sites-available/your-doh-site`:

```nginx
server {
    listen 443 ssl http2;
    server_name doh.yourdomain.com;
    
    # Your existing SSL config...
    
    location /dns-query {
        # Your existing proxy config...
        
        # Add custom log format
        access_log /var/log/nginx/doh-queries.log doh_custom;
    }
}

# Add this at the top level (outside server block)
log_format doh_custom '$remote_addr - [$time_local] "$request" '
                      '$status "$dns_qname" "$dns_qtype"';
```

#### Option B: JSON Format (Easiest to Parse)

```nginx
log_format doh_json escape=json
    '{'
        '"time":"$time_iso8601",'
        '"ip":"$remote_addr",'
        '"domain":"$dns_qname",'
        '"type":"$dns_qtype",'
        '"status":$status'
    '}';

location /dns-query {
    access_log /var/log/nginx/doh-queries.log doh_json;
    # ... rest of config
}
```

#### Option C: Already Have Logs? Use Upstream!

If your nginx just proxies to Pi-hole/Unbound, skip nginx parsing:

```yaml
# In config/config.yaml
doh_logs:
  format: "pihole"  # or "unbound"
  log_paths:
    - "/var/log/pihole/pihole.log"
```

### Step 2: Configure DoH Ad Blocker (1 minute)

Edit `config/config.yaml`:

```yaml
doh_logs:
  format: "nginx_doh"
  log_paths:
    - "/var/log/nginx/doh-queries.log"
    - "/var/log/nginx/doh-queries.log.1"  # rotated logs
  analysis_window_days: 7
  min_query_count: 5

# Everything else can stay default
```

### Step 3: Grant Log Access (if needed)

```bash
# Add your user to adm group (for log access)
sudo usermod -a -G adm $USER

# Or make logs readable
sudo chmod 644 /var/log/nginx/doh-queries.log
```

### Step 4: Test It (30 seconds)

```bash
cd doh-ad-blocker
source venv/bin/activate

# Test log parsing
python tests/test_nginx_doh.py

# Should show:
# ✅ Standard format parsing PASSED
# ✅ JSON format parsing PASSED
# ✅ Classification PASSED
```

### Step 5: Run First Analysis (2 minutes)

```bash
# Run the blocker
python src/main.py

# Check output
ls -lh blocklists/
cat logs/doh-blocker.log
```

## 📊 What Log Format Do I Have?

Run this to check your current nginx logs:

```bash
tail -5 /var/log/nginx/access.log
```

### If you see:
- **Standard format**: `192.168.1.100 - - [15/Jan/2024:10:23:45 +0000] "POST /dns-query..."`
  → Use `format: "nginx_doh"`

- **JSON format**: `{"time":"2024-01-15T10:23:45","domain":"example.com"...}`
  → Use `format: "nginx_doh"` (auto-detects JSON)

- **No DNS queries visible**: Your nginx just proxies
  → Use `format: "pihole"` or `format: "unbound"` and point to upstream logs

## 🔧 Common Nginx DoH Configurations

### Configuration 1: Nginx → doh-server → DNS

```nginx
location /dns-query {
    proxy_pass http://127.0.0.1:8053/dns-query;  # doh-server
    access_log /var/log/nginx/doh.log combined;
}
```

**Use with:** `format: "nginx_doh"` OR read doh-server logs directly

### Configuration 2: Nginx → Pi-hole

```nginx
location /dns-query {
    proxy_pass http://127.0.0.1:8080/dns-query;  # Pi-hole DoH
    access_log /var/log/nginx/doh.log;
}
```

**Use with:** `format: "pihole"` + `/var/log/pihole/pihole.log`

### Configuration 3: Nginx → Unbound

```nginx
upstream dns {
    server 127.0.0.1:53;
}

location /dns-query {
    proxy_pass http://dns;
    access_log /var/log/nginx/doh.log;
}
```

**Use with:** `format: "unbound"` + `/var/log/unbound/unbound.log`

## 🚀 Full Example: My Working Setup

Here's a complete working example:

### /etc/nginx/sites-available/doh

```nginx
server {
    listen 443 ssl http2;
    server_name doh.mynetwork.local;

    ssl_certificate /etc/ssl/certs/doh.crt;
    ssl_certificate_key /etc/ssl/private/doh.key;

    location /dns-query {
        # Proxy to Pi-hole
        proxy_pass http://127.0.0.1:8080/dns-query;
        
        # Don't log in nginx, let Pi-hole log
        access_log off;
        
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

### config/config.yaml

```yaml
doh_logs:
  format: "pihole"
  log_paths:
    - "/var/log/pihole/pihole.log"
    - "/var/log/pihole/pihole.log.1"

youtube:
  enabled: true
  aggressive_mode: true

ml:
  confidence_threshold: 0.85
```

### Result

```bash
$ python src/main.py

Parsed 45,823 queries from 8,234 domains
Identified 43 new ad domains
Generated blocklists in 5 formats
✅ Done!
```

## 🐛 Troubleshooting

### "No domains found"

```bash
# Check if logs exist
ls -la /var/log/nginx/doh*.log

# Check log format
head -1 /var/log/nginx/doh-queries.log

# Verify log permissions
sudo chmod 644 /var/log/nginx/doh-queries.log
```

### "Permission denied"

```bash
# Add user to log group
sudo usermod -a -G adm $(whoami)

# Or run as sudo (not recommended)
sudo python src/main.py
```

### "Parser errors"

```bash
# Test parser directly
python tests/test_nginx_doh.py

# Try different format
# Change format: "nginx_doh" → "pihole"
```

## 📚 Next Steps

1. ✅ Get it working (you're here!)
2. 🤖 Setup automation → See README.md "Automation Settings"
3. 📈 Monitor results → Check `logs/` directory
4. 🔧 Fine-tune → Adjust confidence threshold
5. 🚀 Push to GitHub → Auto-updates daily

## Need Help?

- **Full nginx setup**: See `NGINX_DOH_SETUP.md`
- **Complete docs**: See `README.md`
- **Examples**: See `EXAMPLES.md`
- **Testing**: Run `python tests/test_nginx_doh.py`

---

**TL;DR:**

```bash
# 1. Edit config
nano config/config.yaml
# Set: format: "nginx_doh"
# Set: log_paths: ["/var/log/nginx/doh-queries.log"]

# 2. Run
source venv/bin/activate
python src/main.py

# 3. Done! Check blocklists/
```
