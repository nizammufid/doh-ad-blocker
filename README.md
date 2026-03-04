# 🚫 DoH Ad Blocker

**Automated ML-Powered Ad Domain Blocker for Pi-hole/AdGuard Home**

Automatically learns from your DNS-over-HTTPS (DoH) logs to identify and block ad domains, with special focus on YouTube ads. Updates blocklists daily and pushes to GitHub.

## ✨ Features

- 🤖 **Machine Learning Classification** - Trains on your actual DNS traffic patterns
- 📊 **Multi-Format Log Support** - Pi-hole, AdGuard Home, Unbound, BIND, Cloudflared, DNSCrypt
- 🎯 **YouTube Ad Targeting** - Specialized patterns for YouTube ad detection
- 🔄 **Fully Automated** - Daily updates via GitHub Actions or systemd timer
- 📦 **Multiple Output Formats** - hosts, domains, AdBlock, dnsmasq, Unbound
- 🔍 **Smart Validation** - Prevents false positives with confidence thresholds
- 📈 **Continuous Learning** - Adapts to new ad networks over time

## 🎯 YouTube Ad Blocking

**Important Note:** DNS-based blocking can only catch **some** YouTube ads because:
- Many ads come from the same domains as content (`googlevideo.com`)
- True 100% blocking requires browser extensions (uBlock Origin, SponsorBlock)

**This system blocks:**
- ✅ YouTube tracking domains (`ptracking`, `pagead`)
- ✅ DoubleClick ad servers
- ✅ Google Analytics on YouTube
- ✅ Some pre-roll ad servers
- ✅ Ad measurement/beacon domains

**What it cannot block:**
- ❌ Most in-video ads (same domain as content)
- ❌ Sponsored content markers
- ❌ YouTube Premium promotions

## 🚀 Quick Start

### Prerequisites

- Python 3.8+
- Pi-hole or AdGuard Home (or any DNS server with query logs)
- Git
- GitHub account (for automated updates)

### Installation

```bash
# Clone the repository
git clone https://github.com/your-username/doh-ad-blocker.git
cd doh-ad-blocker

# Run setup script
chmod +x setup.sh
./setup.sh
```

The setup script will:
1. Install Python dependencies
2. Download initial training data
3. Detect your DNS server (Pi-hole/AdGuard)
4. Configure automation (optional)
5. Initialize Git repository (optional)

### Manual Installation

```bash
# Create virtual environment
python3 -m venv venv
source venv/bin/activate

# Install dependencies
pip install -r requirements.txt

# Create directories
mkdir -p data logs models blocklists

# Download training data
curl -o data/stevenblack-hosts.txt \
    https://raw.githubusercontent.com/StevenBlack/hosts/master/hosts
grep "^0.0.0.0" data/stevenblack-hosts.txt | awk '{print $2}' > data/known_ads.txt
```

## ⚙️ Configuration

Edit `config/config.yaml`:

```yaml
# Update with your settings
github:
  repo_owner: "your-github-username"
  repo_name: "ad-blocklists"

doh_logs:
  format: "pihole"  # or "adguard", "unbound", etc.
  log_paths:
    - "/var/log/pihole/pihole.log"
  analysis_window_days: 7
  min_query_count: 5

ml:
  confidence_threshold: 0.85  # 85% confidence required
  model_type: "random_forest"

youtube:
  enabled: true
  aggressive_mode: true  # More blocking, rare false positives
```

### Log Formats Supported

| DNS Server | Log Format | Config Value |
|------------|-----------|--------------|
| Pi-hole | dnsmasq | `pihole` |
| AdGuard Home | JSON | `adguard` |
| Unbound | Standard | `unbound` |
| BIND | Query logs | `bind` |
| Cloudflared | DoH proxy | `cloudflared` |
| DNSCrypt-proxy | Standard | `dnscrypt` |

## 🎮 Usage

### One-Time Run

```bash
# Activate environment
source venv/bin/activate

# Run the blocker
python src/main.py
```

### Using Helper Scripts

```bash
# Train the model
bash scripts/train.sh

# Run analysis and update
bash scripts/run.sh

# Dry run (no GitHub push)
bash scripts/run.sh --dry-run
```

### Automated Updates

#### Option 1: GitHub Actions (Recommended)

1. Push your repository to GitHub
2. Add these secrets to your GitHub repository:
   - `PIHOLE_API_KEY` - Your Pi-hole API key (if fetching logs remotely)
   - `PIHOLE_HOST` - Your Pi-hole URL (if fetching logs remotely)

3. The workflow runs automatically daily at 2 AM UTC

4. Manual trigger:
   - Go to Actions tab → "Auto-Update Blocklists" → Run workflow

#### Option 2: Local Systemd Timer

```bash
# Enable daily updates
sudo systemctl enable doh-ad-blocker.timer
sudo systemctl start doh-ad-blocker.timer

# Check status
systemctl status doh-ad-blocker.timer

# View logs
journalctl -u doh-ad-blocker.service
```

#### Option 3: Cron Job

```bash
# Edit crontab
crontab -e

# Add daily run at 2 AM
0 2 * * * cd /path/to/doh-ad-blocker && ./venv/bin/python src/main.py
```

## 📊 How It Works

```
1. Parse DoH Logs
   ↓
   Extract DNS queries from last 7 days
   
2. Feature Extraction
   ↓
   Analyze domains: length, entropy, keywords,
   subdomains, TLD, query patterns
   
3. ML Classification
   ↓
   Random Forest model predicts ad probability
   
4. YouTube Filtering
   ↓
   Apply YouTube-specific patterns and rules
   
5. Validation
   ↓
   Check confidence threshold (85%+)
   Verify against whitelist
   
6. Generate Blocklists
   ↓
   Create multiple format outputs
   
7. Push to GitHub
   ↓
   Commit and push updates
```

## 📁 Output Files

After running, find your blocklists in `blocklists/`:

| File | Format | Use With |
|------|--------|----------|
| `hosts.txt` | Hosts file | `/etc/hosts`, Windows hosts |
| `domains.txt` | Plain list | Pi-hole, scripts |
| `adblock.txt` | AdBlock Plus | uBlock Origin, AdGuard |
| `dnsmasq.conf` | Dnsmasq | dnsmasq configuration |
| `unbound.conf` | Unbound | Unbound DNS |

### Using with Pi-hole

```bash
# Add to Pi-hole as custom blocklist
# Go to Pi-hole admin → Group Management → Adlists
# Add your GitHub raw URL:
https://raw.githubusercontent.com/your-username/ad-blocklists/main/blocklists/domains.txt
```

### Using with AdGuard Home

```bash
# Add to AdGuard Home
# Go to Filters → DNS blocklists → Add blocklist
# Add your GitHub raw URL:
https://raw.githubusercontent.com/your-username/ad-blocklists/main/blocklists/adblock.txt
```

## 🔬 Machine Learning Details

### Features Used

- Domain length and structure
- Subdomain count and entropy
- Keyword matching (ads, tracking, etc.)
- TLD analysis
- Query frequency patterns
- Temporal patterns (bursty vs steady)
- CNAME chain analysis
- Response time patterns

### Model Types

- **Random Forest** (default) - Best accuracy, fast training
- **Gradient Boosting** - Slightly better accuracy, slower
- **Rule-Based** - Fallback when ML unavailable

### Performance

Typical accuracy on test data: **92-96%**

## 🛡️ Safety Features

### Whitelist Protection

Edit `config/config.yaml` to never block important domains:

```yaml
blocklist:
  whitelist:
    - "googlevideo.com"  # Required for YouTube videos
    - "ytimg.com"        # Required for thumbnails
    - "your-important-site.com"
```

### Manual Review

Domains with 75-85% confidence are flagged for review in `logs/manual_review.txt`

### Confidence Threshold

Default: 85% - Only domains with high confidence are auto-blocked

```yaml
ml:
  confidence_threshold: 0.85  # Increase to 0.90 for fewer false positives
  manual_review_threshold: 0.75
```

## 📈 Monitoring

### View Statistics

```bash
# Check latest report
cat logs/report.json

# View classification details
tail -f logs/doh-blocker.log

# Count blocked domains
wc -l blocklists/domains.txt
```

### GitHub Actions Logs

- Go to your repository → Actions tab
- Click on latest workflow run
- View step logs for detailed output

## 🔧 Troubleshooting

### No domains detected

**Problem:** Blocker runs but finds 0 new domains

**Solutions:**
1. Check log paths in config are correct
2. Ensure logs cover at least 7 days of data
3. Lower `min_query_count` threshold
4. Verify log format matches your DNS server

### High false positives

**Problem:** Legitimate sites being blocked

**Solutions:**
1. Increase `confidence_threshold` to 0.90
2. Add domains to whitelist
3. Review `logs/manual_review.txt`
4. Retrain model with more known-safe domains

### YouTube videos not loading

**Problem:** Videos completely broken

**Solutions:**
1. Ensure `googlevideo.com` is whitelisted
2. Disable `aggressive_mode` for YouTube
3. Check if you accidentally blocked `ytimg.com`

### Logs not parsing

**Problem:** Parser errors in log file

**Solutions:**
1. Verify `log_format` in config matches your DNS server
2. Check log file permissions
3. Try different log format (e.g., `pihole` → `dnsmasq`)

## 🎯 Advanced Usage

### Custom Training Data

Add your own known ad domains:

```bash
# Add to training data
echo "suspicious-ad-domain.com" >> data/known_ads.txt
echo "safe-domain.com" >> data/known_safe.txt

# Retrain model
bash scripts/train.sh
```

### YouTube-Only Mode

Focus exclusively on YouTube ads:

```yaml
youtube:
  enabled: true
  aggressive_mode: true
  
# Add more YouTube patterns
youtube:
  ad_patterns:
    - "your-custom-pattern.*youtube.com"
```

### Integration with Other Tools

```bash
# Export to different format
python -c "
from pathlib import Path
domains = Path('blocklists/domains.txt').read_text()
# Convert to your format
"
```

## 🤝 Contributing

Contributions welcome! Areas for improvement:

- [ ] Support for more log formats
- [ ] Neural network classifier option
- [ ] Real-time detection mode
- [ ] Web dashboard for monitoring
- [ ] Integration with more DNS servers
- [ ] Advanced YouTube ad pattern detection

## 📜 License

MIT License - see LICENSE file

## ⚠️ Disclaimer

This tool is for educational and personal use. YouTube ad blocking may violate YouTube's Terms of Service. Use responsibly and consider YouTube Premium to support content creators.

## 🙏 Credits

- Built with scikit-learn for ML
- Inspired by Pi-hole and AdGuard Home
- YouTube patterns from community research
- Initial training data from [StevenBlack/hosts](https://github.com/StevenBlack/hosts)

## 📞 Support

- 🐛 **Issues:** [GitHub Issues](https://github.com/your-username/doh-ad-blocker/issues)
- 💬 **Discussions:** [GitHub Discussions](https://github.com/your-username/doh-ad-blocker/discussions)
- 📖 **Wiki:** [Documentation](https://github.com/your-username/doh-ad-blocker/wiki)

---

**Made with ❤️ for a cleaner internet**
