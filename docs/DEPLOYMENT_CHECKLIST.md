# ✅ Deployment Checklist - Lone Star Chat v1.0.0

**Complete pre-deployment, deployment, and post-deployment checklist**

---

## 📋 Pre-Deployment Checklist

### Infrastructure Preparation

#### Server Hardware
- [ ] Server acquired (16+ cores, 64+ GB RAM, 500+ GB SSD)
- [ ] Ubuntu Server 22.04 LTS installed
- [ ] Server has internet connectivity
- [ ] Static IP address configured (or DHCP reservation)
- [ ] Server is accessible via SSH
- [ ] Root access available

#### Network Requirements
- [ ] Ports 80, 443 open in router/firewall (for external access)
- [ ] Port 22 open for SSH (restricted to admin IPs if possible)
- [ ] Domain name configured (optional, can use IP)
- [ ] DNS records pointing to server (if using domain)

#### Credentials Prepared
- [ ] Admin email decided
- [ ] Strong admin password ready (12+ characters)
- [ ] Database password ready (random, 32+ characters)
- [ ] Redis password ready (random, 32+ characters)
- [ ] Meilisearch master key ready (random, 32+ characters)
- [ ] JWT secret ready (random, 64+ characters)

---

## 🚀 Deployment Checklist

### Phase 1: Initial Setup (10 minutes)

- [ ] SSH into server as root
- [ ] Copy project to `/srv/Lone Star Chat/`
- [ ] Verify all files present: `ls -la /srv/Lone\ Star\ Chat/`
- [ ] Make scripts executable: `chmod +x /srv/Lone\ Star\ Chat/scripts/*.sh`

### Phase 2: Environment Configuration (5 minutes)

- [ ] Copy `.env.example` to `.env`: `cp .env.example .env`
- [ ] Edit `.env` file: `nano /srv/Lone\ Star\ Chat/.env`
- [ ] Set all passwords and secrets (remove all `CHANGE_ME` values)
- [ ] Configure `ADMIN_EMAIL` and `ADMIN_PASSWORD`
- [ ] Set `NODE_ENV=production`
- [ ] Save and exit

### Phase 3: Installation (15-30 minutes)

- [ ] Run installation script: `sudo bash /srv/Lone\ Star\ Chat/scripts/install.sh`
- [ ] Wait for completion (watch for errors)
- [ ] Verify installation completed successfully
- [ ] Note generated passwords if any were auto-generated

### Phase 4: LLaMA Model Download (10-60 minutes, depends on internet)

- [ ] Check if model exists: `ls -lh /srv/Lone\ Star\ Chat/models/`
- [ ] If not present, run: `bash /srv/Lone\ Star\ Chat/scripts/download-model.sh`
- [ ] Verify model size is ~4.5 GB
- [ ] Wait for download to complete

### Phase 5: Service Startup (5 minutes)

- [ ] Start all services: `cd /srv/Lone\ Star\ Chat && docker-compose up -d`
- [ ] Wait 2-3 minutes for services to initialize
- [ ] Check service status: `docker-compose ps`
- [ ] All services should show "Up" status

---

## 🔍 Validation Checklist

### Automated Validation

- [ ] Run validation script: `sudo bash /srv/Lone\ Star\ Chat/scripts/validate.sh`
- [ ] Review validation results
- [ ] All checks should pass (green ✓)
- [ ] Address any warnings (yellow ⚠)
- [ ] Fix any failures (red ✗)

### Manual Service Checks

#### Docker Services
- [ ] PostgreSQL: `docker exec lonestar-postgres pg_isready`
- [ ] Redis: `docker exec lonestar-redis redis-cli ping` (should return PONG)
- [ ] Meilisearch: `curl http://localhost:7700/health` (should return OK)
- [ ] LLaMA: `curl http://localhost:8080/health` (should return OK, may take 5-10 min)
- [ ] API: `curl http://localhost:3000/api/health` (should return OK)
- [ ] Nginx: `curl -k https://localhost/api/health` (should return OK)

#### Database Tables
- [ ] Connect to database: `docker exec -it lonestar-postgres psql -U lonestar -d lonestar_db`
- [ ] List tables: `\dt`
- [ ] Should see 10 tables: users, chats, chat_members, messages, schedules, etc.
- [ ] Check admin user: `SELECT email FROM users WHERE role='admin';`
- [ ] Exit: `\q`

#### Logs Review
- [ ] Check API logs: `docker-compose -f /srv/Lone\ Star\ Chat/docker-compose.yml logs api | tail -50`
- [ ] Check LLaMA logs: `docker-compose -f /srv/Lone\ Star\ Chat/docker-compose.yml logs llama | tail -50`
- [ ] No critical errors should be present

---

## 🔐 Security Hardening Checklist

### Firewall Configuration
- [ ] UFW is active: `sudo ufw status`
- [ ] Port 22 open (SSH)
- [ ] Port 80 open (HTTP redirect to HTTPS)
- [ ] Port 443 open (HTTPS)
- [ ] All other ports closed

### Fail2Ban
- [ ] Fail2Ban is active: `sudo systemctl status fail2ban`
- [ ] SSH jail enabled
- [ ] Nginx jails enabled (if configured)

### SSL/TLS Certificates

#### Option A: Let's Encrypt (Recommended for production with domain)
- [ ] Domain name pointing to server
- [ ] Port 80 accessible from internet
- [ ] Run: `sudo certbot --nginx -d yourdomain.com`
- [ ] Certificate obtained successfully
- [ ] Auto-renewal configured

#### Option B: Self-Signed (For internal/testing)
- [ ] Generate certificate: `sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/lonestar.key -out /etc/ssl/certs/lonestar.crt`
- [ ] Update nginx config to use self-signed cert
- [ ] Restart nginx: `docker-compose restart nginx`

### Password Security
- [ ] All default passwords changed in `.env`
- [ ] Admin password is strong (12+ characters)
- [ ] Database passwords are random and complex
- [ ] JWT secret is random and long (64+ characters)
- [ ] No passwords stored in plain text files

### SSH Security
- [ ] Root login disabled (optional but recommended)
- [ ] Password authentication disabled (optional, use key-based)
- [ ] SSH port changed (optional)

---

## 📱 Mobile App Configuration Checklist

### iOS Build

- [ ] Xcode installed (macOS required)
- [ ] Navigate to mobile directory: `cd /srv/Lone\ Star\ Chat/mobile`
- [ ] Update API URL in `lib/config/api_config.dart` to your server's URL
- [ ] Install dependencies: `flutter pub get`
- [ ] Build for iOS: `flutter build ios --release`
- [ ] Open in Xcode: `open ios/Runner.xcworkspace`
- [ ] Configure signing (Apple Developer account required)
- [ ] Archive and upload to App Store Connect or TestFlight
- [ ] Submit for review (if publishing to App Store)

### Android Build

- [ ] Navigate to mobile directory: `cd /srv/Lone\ Star\ Chat/mobile`
- [ ] Update API URL in `lib/config/api_config.dart` to your server's URL
- [ ] Install dependencies: `flutter pub get`
- [ ] Build APK: `flutter build apk --release`
- [ ] APK location noted: `build/app/outputs/flutter-apk/app-release.apk`
- [ ] (Optional) Build AAB for Play Store: `flutter build appbundle --release`
- [ ] Test APK on device
- [ ] Sign APK (if publishing)
- [ ] Distribute to employees

### App Configuration

- [ ] Video splash screen present: `mobile/assets/videos/Lone Star Chat.mp4`
- [ ] Base URL configured correctly (https://YOUR_DOMAIN or https://SERVER_IP)
- [ ] App version set: `pubspec.yaml` version field
- [ ] App name and bundle ID configured

---

## 🎛️ Admin Panel Setup Checklist

### Initial Login

- [ ] Open browser to: `https://YOUR_DOMAIN/admin` or `https://SERVER_IP/admin`
- [ ] Accept SSL warning if using self-signed certificate
- [ ] Login with admin credentials from `.env`
- [ ] Change admin password immediately

### User Management

- [ ] Create user accounts for all employees
- [ ] Assign appropriate roles (admin, manager, employee)
- [ ] Assign departments
- [ ] Send credentials securely to users

### App Settings

- [ ] Set iOS app version (e.g., 1.0.0)
- [ ] Set Android app version (e.g., 1.0.0)
- [ ] Configure force update (on/off)
- [ ] Set minimum supported versions
- [ ] Configure file upload limits

### RAG Documents

- [ ] Upload company policies (PDF, DOCX)
- [ ] Upload vehicle specifications
- [ ] Upload training materials
- [ ] Upload FAQs
- [ ] Verify documents are indexed (check Meilisearch)

---

## 🔄 Backup & Monitoring Setup Checklist

### Automated Backups

- [ ] Run cron setup script: `sudo bash /srv/Lone\ Star\ Chat/scripts/setup-cron.sh`
- [ ] Verify cron jobs added: `crontab -l`
- [ ] Should see backup job (daily at 2 AM)
- [ ] Should see monitoring job (every 5 minutes)
- [ ] Test manual backup: `sudo bash /srv/Lone\ Star\ Chat/scripts/backup.sh`
- [ ] Verify backup created in `/srv/Lone Star Chat/backups/`
- [ ] Check backup size (should be 100MB-1GB depending on data)

### Backup Storage

- [ ] Configure external backup storage (optional but recommended)
- [ ] Set up offsite backup replication (optional)
- [ ] Document backup restore procedure
- [ ] Test restore on separate server (highly recommended)

### Monitoring

- [ ] Run monitoring script: `sudo bash /srv/Lone\ Star\ Chat/scripts/monitor.sh`
- [ ] Review output (all services should be healthy)
- [ ] Configure alerting (optional, via email or webhook)
- [ ] Set up log rotation
- [ ] Configure external monitoring (optional, e.g., Uptime Robot)

---

## 📊 Performance Tuning Checklist

### For 100+ Users

- [ ] Increase nginx worker processes in `nginx/nginx.conf`:
  ```nginx
  worker_processes auto;
  worker_connections 4096;
  ```

- [ ] Increase PostgreSQL connection pool in `.env`:
  ```
  DB_POOL_SIZE=50
  DB_MAX_CONNECTIONS=200
  ```

- [ ] Increase Redis maxmemory in `docker-compose.yml`:
  ```yaml
  command: redis-server --maxmemory 8gb --maxmemory-policy allkeys-lru
  ```

- [ ] Optimize LLaMA threads in `docker-compose.yml`:
  ```yaml
  command: -m /models/llama-3-8b-instruct-q4_k_m.gguf -c 4096 -n 512 --threads 16
  ```

- [ ] Restart services: `docker-compose restart`

---

## 👥 User Onboarding Checklist

### Prepare Training Materials

- [ ] Print quick reference cards
- [ ] Schedule training sessions (3 weeks)
- [ ] Assign trainers
- [ ] Prepare quiz and practical assessment

### Distribute App

#### Android
- [ ] Copy APK to shared drive or email
- [ ] Provide installation instructions
- [ ] Help users enable "Unknown Sources"
- [ ] Verify app installs successfully

#### iOS
- [ ] Enroll in Apple Developer Program ($99/year)
- [ ] Upload to TestFlight
- [ ] Invite employees via email
- [ ] Guide through TestFlight installation

### User Accounts

- [ ] Create user accounts for all employees
- [ ] Generate temporary passwords
- [ ] Send credentials securely (not via email if possible)
- [ ] Instruct users to change password on first login

### Training Schedule

- [ ] Week 1: Basic navigation (all users)
- [ ] Week 2: Advanced features (all users)
- [ ] Week 3: Best practices (all users)
- [ ] Manager training: Admin panel usage (managers only)

---

## 📋 Post-Deployment Checklist

### Day 1

- [ ] Monitor logs continuously: `docker-compose logs -f`
- [ ] Check for errors or warnings
- [ ] Verify users can login
- [ ] Test sending messages
- [ ] Test AI assistant
- [ ] Test voice calls (at least 2 users)
- [ ] Monitor system resources: `htop`

### Week 1

- [ ] Daily log review
- [ ] Monitor disk space: `df -h`
- [ ] Check backup completion (verify backup files exist)
- [ ] Gather user feedback
- [ ] Address any reported issues
- [ ] Monitor system performance

### Month 1

- [ ] Weekly system updates: `sudo bash /srv/Lone\ Star\ Chat/scripts/update.sh`
- [ ] Test backup restore procedure
- [ ] Review user adoption metrics
- [ ] Conduct user satisfaction survey
- [ ] Identify feature requests
- [ ] Plan for v1.1.0 updates

### Ongoing

- [ ] Daily: Monitor health, backups
- [ ] Weekly: Security updates, log review
- [ ] Monthly: Backup restore test, performance review
- [ ] Quarterly: Full system audit, documentation update

---

## 🐛 Troubleshooting Checklist

### If Services Won't Start

- [ ] Check Docker status: `systemctl status docker`
- [ ] Check logs: `docker-compose logs`
- [ ] Check disk space: `df -h`
- [ ] Check memory: `free -h`
- [ ] Restart Docker: `systemctl restart docker`
- [ ] Restart services: `docker-compose restart`

### If AI Is Slow

- [ ] Check CPU usage: `htop`
- [ ] Check LLaMA logs: `docker-compose logs llama`
- [ ] Verify model loaded: Look for "loaded" in logs
- [ ] Increase threads in `docker-compose.yml`
- [ ] Consider using Q4_0 quantization (smaller, faster)

### If Users Can't Connect

- [ ] Check firewall: `sudo ufw status`
- [ ] Check nginx: `docker-compose ps nginx`
- [ ] Check SSL certificate: `curl -I https://YOUR_DOMAIN`
- [ ] Check API: `curl http://localhost:3000/api/health`
- [ ] Review nginx logs: `docker-compose logs nginx`

### If Database Issues

- [ ] Check PostgreSQL: `docker exec lonestar-postgres pg_isready`
- [ ] Check connections: `docker exec lonestar-postgres psql -U lonestar -d lonestar_db -c "SELECT count(*) FROM pg_stat_activity;"`
- [ ] Check disk space: `df -h`
- [ ] Review PostgreSQL logs: `docker-compose logs postgres`

---

## ✅ Final Verification

### Before Going Live

- [ ] All services running and healthy
- [ ] All backups configured and tested
- [ ] All user accounts created
- [ ] Mobile apps built and distributed
- [ ] Training scheduled
- [ ] Documentation accessible to admins
- [ ] Emergency contacts documented
- [ ] Rollback plan prepared

### Sign-Off

- [ ] IT Department approval
- [ ] Management approval
- [ ] Legal/HR compliance reviewed
- [ ] Users notified of go-live date
- [ ] Support procedures documented

### Go-Live

- [ ] Announce to all employees
- [ ] Monitor closely for first 48 hours
- [ ] Collect feedback
- [ ] Address issues immediately
- [ ] Celebrate success! 🎉

---

## 📞 Support Contacts

| Issue Type | Contact |
|------------|---------|
| 🔧 Technical Issues | IT Department / System Admin |
| 🔐 Security Concerns | Security Officer |
| 👥 User Issues | Help Desk |
| 📚 Training Questions | HR / Training Team |
| ⚖️ Legal/Compliance | Legal Counsel |

---

## 📝 Notes Section

Use this space to document any issues, customizations, or important information specific to your deployment:

```
Deployment Date: _______________
Server IP: _______________
Domain Name: _______________
Admin Email: _______________
Notes:
_________________________________
_________________________________
_________________________________
_________________________________
```

---

**Checklist Version**: 1.0.0  
**Last Updated**: 2025-01-04  
**Status**: Ready for Production Deployment 🚀

---

<div align="center">

**Follow this checklist carefully for a successful deployment!**

**Good luck! 🍀**

</div>
