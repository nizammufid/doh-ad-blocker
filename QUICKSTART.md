# 🚀 Quick Start Guide

## Get Up and Running in 5 Minutes

### Step 1: Install (2 minutes)

```bash
# Clone and setup
git clone https://github.com/your-username/doh-ad-blocker.git
cd doh-ad-blocker
chmod +x setup.sh
./setup.sh
```

### Step 2: Configure (1 minute)

Edit `config/config.yaml`:

```yaml
# Required changes:
github:
  repo_owner: "YOUR_GITHUB_USERNAME"  # ← Change this
  repo_name: "ad-blocklists"          # ← Change this

# Optional but recommended:
doh_logs:
  log_paths:
    - "/var/log/pihole/pihole.log"    # ← Verify path is correct
```

### Step 3: Test (1 minute)

```bash
# Activate environment
source venv/bin/activate

# Run system tests
python tests/test_system.py
```

Expected output:
```
✅ Log Parser Test PASSED
✅ ML Classifier Test PASSED
✅ YouTube Detection Test PASSED
✅ Model Training Test PASSED
🎉 All Tests Completed!
```

### Step 4: First Run (1 minute)

```bash
# Run the blocker
python src/main.py
```

Check output in `blocklists/` directory!

---

## What Just Happened?

1. ✅ Parsed your DNS logs (last 7 days)
2. ✅ Trained ML model on 125k+ known ad domains
3. ✅ Classified new domains with 85%+ confidence
4. ✅ Applied YouTube-specific ad patterns
5. ✅ Generated blocklists in 5 formats

## Next: Automate It!

### Option A: GitHub Actions (Cloud, Free)

```bash
# 1. Create GitHub repo
# 2. Push code
git remote add origin https://github.com/YOUR_USERNAME/ad-blocklists.git
git push -u origin main

# 3. Runs automatically daily at 2 AM UTC!
```

### Option B: Local Cron (On your Pi-hole server)

```bash
crontab -e
# Add: 0 2 * * * cd /path/to/doh-ad-blocker && ./venv/bin/python src/main.py
```

### Option C: Systemd Timer (Recommended for Linux)

```bash
sudo systemctl enable doh-ad-blocker.timer
sudo systemctl start doh-ad-blocker.timer
```

## Using Your Blocklists

### In Pi-hole

1. Go to **Settings → Adlists**
2. Add: `https://raw.githubusercontent.com/YOUR_USERNAME/ad-blocklists/main/blocklists/domains.txt`
3. Click **Update Gravity**

### In AdGuard Home

1. Go to **Filters → DNS blocklists**
2. Add: `https://raw.githubusercontent.com/YOUR_USERNAME/ad-blocklists/main/blocklists/adblock.txt`
3. Click **Save**

---

## Troubleshooting

**"No domains detected"**
- Check log path in config is correct
- Ensure 7+ days of logs exist

**"scikit-learn not found"**
- Run: `pip install scikit-learn numpy --break-system-packages`

**YouTube videos broken**
- Add to whitelist: `googlevideo.com`, `ytimg.com`

**Need help?**
- Check full [README.md](README.md)
- See [EXAMPLES.md](EXAMPLES.md) for sample output

---

## What's Blocked?

### ✅ Will Block (DNS Level)
- Ad tracking pixels
- Analytics beacons
- DoubleClick ad servers
- Google Analytics (on YouTube)
- Some YouTube pre-roll ads
- Telemetry domains

### ❌ Cannot Block (Requires Browser Extension)
- Most YouTube in-video ads
- Sponsored content markers
- Same-domain ad delivery

**For 100% YouTube ad blocking:** Use **uBlock Origin** browser extension!

---

**You're all set! 🎉**

Your system will now learn from your DNS traffic and automatically update blocklists daily.
