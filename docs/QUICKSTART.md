# 🚀 Quick Start Guide - Lone Star Chat

## One-Command Installation

```bash
sudo bash /srv/Lone\ Star\ Chat/scripts/install.sh
```

That's it! The script will automatically:
- ✅ Install all dependencies
- ✅ Configure firewall
- ✅ Download AI model
- ✅ Generate secure passwords
- ✅ Start all services
- ✅ Run health checks

---

## Post-Installation (5 Minutes)

### 1. Get Admin Credentials

```bash
cd /srv/Lone\ Star\ Chat
cat .env | grep -E "ADMIN_EMAIL|ADMIN_PASSWORD"
```

**Default:**
- Email: `admin@lonestar.local`
- Password: `admin123` (change immediately!)

### 2. Access Admin Panel

Open browser: `https://YOUR_SERVER_IP/admin`

**First Steps:**
1. Change admin password
2. Create user accounts
3. Upload company documents for AI
4. Set app version (iOS/Android)
5. Enable force update if needed

### 3. Configure Mobile App

Edit mobile app config:

```bash
nano /srv/Lone\ Star\ Chat/mobile/lib/config/api_config.dart
```

Change:
```dart
static const String baseUrl = 'https://YOUR_DOMAIN_OR_IP';
```

Build apps:
```bash
cd /srv/Lone\ Star\ Chat/mobile

# iOS
flutter build ios --release

# Android
flutter build apk --release
```

APK location: `build/app/outputs/flutter-apk/app-release.apk`

### 4. Distribute to Employees

**Option A: Direct APK (Android)**
- Copy APK to shared drive
- Employees install manually
- Enable "Unknown Sources" in Android settings

**Option B: Internal App Store**
- Use MDM solution (like Microsoft Intune)
- Deploy via corporate app store
- Automatic updates available

**Option C: TestFlight (iOS)**
- Enroll in Apple Developer Program ($99/year)
- Upload to TestFlight
- Invite employees via email

---

## Daily Operations

### Monitor System

```bash
sudo bash /srv/Lone\ Star\ Chat/scripts/monitor.sh
```

Shows:
- ✅ Service status
- 💾 Disk usage
- 🧠 Memory usage
- 🔌 CPU load
- 📊 Database connections

### View Logs

```bash
# All services
docker-compose -f /srv/Lone\ Star\ Chat/docker-compose.yml logs -f

# Specific service
docker-compose -f /srv/Lone\ Star\ Chat/docker-compose.yml logs -f api

# Error logs only
docker-compose -f /srv/Lone\ Star\ Chat/docker-compose.yml logs -f api | grep ERROR
```

### Backup

Automatic daily backups at 2 AM (configured during installation).

Manual backup:
```bash
sudo bash /srv/Lone\ Star\ Chat/scripts/backup.sh
```

Backups stored in: `/srv/Lone Star Chat/backups/`

### Restore from Backup

```bash
sudo bash /srv/Lone\ Star\ Chat/scripts/restore.sh /srv/Lone\ Star\ Chat/backups/backup-2025-01-04.tar.gz
```

### Update System

```bash
sudo bash /srv/Lone\ Star\ Chat/scripts/update.sh
```

Automatically:
- Creates backup
- Pulls latest changes
- Restarts services
- Runs health checks

---

## Common Tasks

### Add New User

**Via Admin Panel:**
1. Login to admin panel
2. Navigate to Users → Add New
3. Fill in details
4. Click "Create User"
5. Share credentials securely

**Via API:**
```bash
curl -X POST https://YOUR_DOMAIN/api/users \
  -H "Authorization: Bearer ADMIN_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "email": "newuser@lonestar.local",
    "password": "secure123",
    "first_name": "John",
    "last_name": "Doe",
    "role": "employee",
    "department": "Sales"
  }'
```

### Upload Document for AI

**Via Admin Panel:**
1. Go to AI → Documents
2. Click "Upload"
3. Select file (PDF, DOCX, etc.)
4. Add title and category
5. Wait for processing (30-60 seconds)

**Supported formats:**
- PDF documents
- Word (DOCX)
- Excel (XLSX)
- Plain text (TXT)
- Markdown (MD)

### Force App Update

1. Login to admin panel
2. Go to Settings → App Versions
3. Change version numbers
4. Toggle "Force Update" ON
5. Save changes

Next time users open app, they'll be prompted to update.

### Disable User Account

**Via Admin Panel:**
1. Users → Find user
2. Click "Edit"
3. Set "Active" to OFF
4. Save

User will be logged out immediately.

### Reset User Password

**Via Admin Panel:**
1. Users → Find user
2. Click "Reset Password"
3. Copy temporary password
4. Share securely with user

**Via CLI:**
```bash
docker exec -it lonestar-api node scripts/reset-password.js user@lonestar.local
```

### View System Metrics

**Real-time monitoring:**
```bash
# CPU, Memory, Network
htop

# Docker stats
docker stats

# Disk usage
df -h
```

**Historical data:**
- Check logs: `/srv/Lone Star Chat/logs/`
- Database size: Check admin panel

---

## Troubleshooting

### Service Won't Start

```bash
# Check service status
docker-compose -f /srv/Lone\ Star\ Chat/docker-compose.yml ps

# Restart specific service
docker-compose -f /srv/Lone\ Star\ Chat/docker-compose.yml restart api

# Restart all
docker-compose -f /srv/Lone\ Star\ Chat/docker-compose.yml restart
```

### AI Not Responding

```bash
# Check LLaMA service
docker-compose -f /srv/Lone\ Star\ Chat/docker-compose.yml logs llama

# Restart LLaMA
docker-compose -f /srv/Lone\ Star\ Chat/docker-compose.yml restart llama

# Test directly
curl http://localhost:8080/health
```

### Database Issues

```bash
# Check PostgreSQL logs
docker-compose -f /srv/Lone\ Star\ Chat/docker-compose.yml logs postgres

# Access database
docker exec -it lonestar-postgres psql -U lonestar -d lonestar_db

# Run SQL: \dt (list tables), \q (quit)
```

### Out of Disk Space

```bash
# Check usage
df -h

# Clean Docker
docker system prune -a --volumes

# Delete old backups (keep last 7)
find /srv/Lone\ Star\ Chat/backups/ -name "backup-*.tar.gz" -mtime +7 -delete

# Clean logs
docker-compose -f /srv/Lone\ Star\ Chat/docker-compose.yml logs --tail=0 -f
```

### Performance Issues

```bash
# Check system resources
htop

# Database connections
docker exec lonestar-postgres psql -U lonestar -d lonestar_db -c "SELECT count(*) FROM pg_stat_activity;"

# Restart services
docker-compose -f /srv/Lone\ Star\ Chat/docker-compose.yml restart
```

### Users Can't Login

1. Check service status
2. Verify credentials
3. Check network connectivity
4. Review error logs
5. Try password reset

### SSL Certificate Issues

```bash
# Renew Let's Encrypt
certbot renew

# Check expiration
certbot certificates

# Test SSL
curl -I https://YOUR_DOMAIN
```

---

## Security Checklist

- [ ] Changed default admin password
- [ ] Firewall enabled (UFW)
- [ ] Fail2Ban configured
- [ ] SSL/TLS enabled
- [ ] Regular backups running
- [ ] Monitoring active
- [ ] User passwords strong (8+ chars)
- [ ] Role-based access configured
- [ ] Audit logs reviewed monthly

---

## Performance Tips

### For 100+ Users

**Hardware:**
- CPU: Ryzen 9 / Xeon (8+ cores)
- RAM: 64-128 GB
- Storage: 1 TB SSD
- Network: 1 Gbps

**Configuration:**

1. **Increase worker processes** (`/srv/Lone Star Chat/nginx/nginx.conf`):
```nginx
worker_processes auto;
worker_connections 4096;
```

2. **Database tuning** (`.env`):
```
DB_POOL_SIZE=50
DB_MAX_CONNECTIONS=200
```

3. **Redis caching**:
```
REDIS_CACHE_TTL=3600
```

4. **LLaMA optimization** (`docker-compose.yml`):
```yaml
command: -m /models/llama-3-8b-instruct-q4_k_m.gguf -c 4096 -n 512 --threads 8
```

### Monitoring

Enable system monitoring:
```bash
# Add to cron (every 5 minutes)
*/5 * * * * /srv/Lone\ Star\ Chat/scripts/monitor.sh >> /var/log/lonestar-monitor.log 2>&1
```

---

## Backup Strategy

### Automated (Recommended)

Daily backups at 2 AM via cron (configured during installation).

### Manual Schedule

- **Daily**: Database + uploads (fast, 5-10 minutes)
- **Weekly**: Full system backup (slower, 30-60 minutes)
- **Monthly**: Archive to external storage

### Retention Policy

- Keep daily backups: 7 days
- Keep weekly backups: 4 weeks
- Keep monthly backups: 12 months

### Test Restores

**Monthly:**
```bash
# Restore to test environment
sudo bash /srv/Lone\ Star\ Chat/scripts/restore.sh /srv/Lone\ Star\ Chat/backups/backup-latest.tar.gz
```

---

## Scaling

### Vertical (Single Server)

Increase resources:
- More CPU cores → faster AI
- More RAM → larger model / more users
- Faster SSD → better database performance

### Horizontal (Multiple Servers)

**Load Balancing:**
1. Deploy multiple API servers
2. Use nginx as load balancer
3. Shared PostgreSQL / Redis / Meilisearch

**Architecture:**
```
[Load Balancer] → [API-1] → [Shared DB]
                → [API-2] ↗
                → [API-3]
```

---

## Update Schedule

### Security Updates
**Frequency:** Weekly  
**Process:**
```bash
sudo apt update && sudo apt upgrade -y
sudo bash /srv/Lone\ Star\ Chat/scripts/update.sh
```

### Feature Updates
**Frequency:** Monthly  
**Process:**
1. Review changelog
2. Test in staging
3. Schedule maintenance window
4. Run update script
5. Verify functionality

### AI Model Updates
**Frequency:** Quarterly  
**Process:**
1. Download new model
2. Update docker-compose.yml
3. Restart llama service
4. Test AI responses

---

## Support Resources

### Documentation
- Architecture: `/srv/Lone Star Chat/docs/ARCHITECTURE.md`
- API Reference: `/srv/Lone Star Chat/docs/API.md`
- Deployment: `/srv/Lone Star Chat/docs/DEPLOYMENT.md`
- Legal: `/srv/Lone Star Chat/docs/LEGAL.md`
- Training: `/srv/Lone Star Chat/docs/TRAINING.md`

### Logs
- Application: `docker-compose logs -f api`
- Database: `docker-compose logs -f postgres`
- AI: `docker-compose logs -f llama`
- Nginx: `/var/log/nginx/`

### Health Checks
- API: `https://YOUR_DOMAIN/api/health`
- Database: `docker exec lonestar-postgres pg_isready`
- Redis: `docker exec lonestar-redis redis-cli ping`
- LLaMA: `curl http://localhost:8080/health`

---

## Emergency Contacts

| Issue | Contact |
|-------|---------|
| System Down | IT Department / On-Call Admin |
| Security Breach | Security Officer + Legal |
| Data Loss | Backup Administrator |
| Performance Issues | System Administrator |
| User Issues | Help Desk |

---

## Quick Commands Reference

```bash
# Start all services
docker-compose -f /srv/Lone\ Star\ Chat/docker-compose.yml up -d

# Stop all services
docker-compose -f /srv/Lone\ Star\ Chat/docker-compose.yml down

# Restart services
docker-compose -f /srv/Lone\ Star\ Chat/docker-compose.yml restart

# View logs (real-time)
docker-compose -f /srv/Lone\ Star\ Chat/docker-compose.yml logs -f

# Check status
docker-compose -f /srv/Lone\ Star\ Chat/docker-compose.yml ps

# Backup
sudo bash /srv/Lone\ Star\ Chat/scripts/backup.sh

# Restore
sudo bash /srv/Lone\ Star\ Chat/scripts/restore.sh /path/to/backup.tar.gz

# Monitor
sudo bash /srv/Lone\ Star\ Chat/scripts/monitor.sh

# Update
sudo bash /srv/Lone\ Star\ Chat/scripts/update.sh
```

---

**Need Help?**  
📧 Email: support@lonestar.local  
📖 Docs: `/srv/Lone Star Chat/docs/`  
🐛 Issues: Check logs and review troubleshooting guide

**System installed successfully? Start here:**
1. Change admin password ✅
2. Create users ✅
3. Build mobile app ✅
4. Upload documents ✅
5. Train employees 📚

---

**Version**: 1.0.0  
**Last Updated**: 2025-10-04  
**Status**: Production Ready 🚀
