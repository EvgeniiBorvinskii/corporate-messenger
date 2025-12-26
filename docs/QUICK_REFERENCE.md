# 📋 Lone Star Chat - Quick Reference Card

**Print this page for quick access to essential information**

---

## 🚀 One-Page System Overview

### What Is It?
**Lone Star Chat** - Private, local communication platform for 100 employees with AI assistant

### Key Features
- ✅ 100% Private (all data local)
- ✅ AI Assistant (LLaMA 3 8B with RAG)
- ✅ Real-time messaging
- ✅ Voice calls
- ✅ Calendar/scheduling
- ✅ Mobile app (iOS/Android)

---

## 📍 Server Information

### Location
```
/srv/Lone Star Chat/
```

### Admin Access
```
URL: https://YOUR_DOMAIN/admin
Email: admin@lonestar.local
Password: [CHECK .env FILE]
```

### Server Requirements
- CPU: 16+ cores
- RAM: 64+ GB
- Storage: 500+ GB SSD
- OS: Ubuntu Server 22.04

---

## ⚡ Essential Commands

### Start System
```bash
cd /srv/Lone\ Star\ Chat
docker-compose up -d
```

### Stop System
```bash
docker-compose down
```

### Check Status
```bash
docker-compose ps
bash scripts/monitor.sh
```

### View Logs
```bash
docker-compose logs -f api
docker-compose logs -f llama
```

### Backup Now
```bash
sudo bash scripts/backup.sh
```

### Restore Backup
```bash
sudo bash scripts/restore.sh /path/to/backup.tar.gz
```

### Update System
```bash
sudo bash scripts/update.sh
```

### Validate Installation
```bash
sudo bash scripts/validate.sh
```

---

## 🔧 Common Tasks

### Add New User (via API)
```bash
curl -X POST https://YOUR_DOMAIN/api/users \
  -H "Authorization: Bearer ADMIN_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"email":"user@lonestar.local","password":"pass123","first_name":"John","last_name":"Doe","role":"employee"}'
```

### Reset Password (via CLI)
```bash
docker exec -it lonestar-api node scripts/reset-password.js user@lonestar.local
```

### Upload RAG Document
Via Admin Panel → AI → Documents → Upload

### Enable Force Update
Via Admin Panel → Settings → Force Update → ON

### Check Database
```bash
docker exec -it lonestar-postgres psql -U lonestar -d lonestar_db
\dt      # List tables
\q       # Quit
```

---

## 🚨 Troubleshooting Quick Fixes

### Service Won't Start
```bash
docker-compose restart SERVICE_NAME
# Example: docker-compose restart api
```

### AI Not Responding
```bash
docker-compose restart llama
# Wait 5-10 minutes for model to load
```

### Database Connection Error
```bash
docker-compose restart postgres
docker-compose restart api
```

### Out of Disk Space
```bash
# Clean Docker
docker system prune -a

# Delete old backups
find /srv/Lone\ Star\ Chat/backups/ -name "backup-*.tar.gz" -mtime +7 -delete
```

### Check System Resources
```bash
htop            # CPU/Memory
df -h           # Disk space
docker stats    # Container resources
```

---

## 📊 Health Check URLs

### API Server
```
http://localhost:3000/api/health
```

### PostgreSQL
```bash
docker exec lonestar-postgres pg_isready
```

### Redis
```bash
docker exec lonestar-redis redis-cli ping
```

### Meilisearch
```
http://localhost:7700/health
```

### LLaMA
```
http://localhost:8080/health
```

### Nginx
```
https://YOUR_DOMAIN/api/health
```

---

## 📁 Important Files

| File | Purpose |
|------|---------|
| `/srv/Lone Star Chat/.env` | Configuration |
| `/srv/Lone Star Chat/docker-compose.yml` | Services |
| `/srv/Lone Star Chat/logs/` | Application logs |
| `/srv/Lone Star Chat/backups/` | Backup files |
| `/srv/Lone Star Chat/models/` | AI model |

---

## 📚 Documentation Quick Links

| Document | Purpose |
|----------|---------|
| `README.md` | Overview |
| `QUICKSTART.md` | Deploy in 5 min |
| `docs/ARCHITECTURE.md` | System design |
| `docs/API.md` | API reference |
| `docs/DEPLOYMENT.md` | Full deployment |
| `docs/TRAINING.md` | User training |
| `docs/LEGAL.md` | Compliance |

---

## 🔐 Security Checklist

- [ ] All default passwords changed
- [ ] UFW firewall active (ports 22, 80, 443)
- [ ] Fail2Ban running
- [ ] SSL certificate installed
- [ ] Backups running daily (2 AM)
- [ ] Logs reviewed weekly
- [ ] Users trained on security

---

## 📅 Maintenance Schedule

### Daily
- Automated backup (2 AM)
- Health monitoring (every 5 min)

### Weekly
- Security updates: `apt update && apt upgrade`
- Log review: `docker-compose logs`
- Disk space check: `df -h`

### Monthly
- Test backup restore
- Review user activity
- Performance tuning

### Quarterly
- Full system audit
- Documentation review
- Feature updates

---

## 📞 Emergency Contacts

| Issue | Contact |
|-------|---------|
| 🔧 System Down | IT Department |
| 🔐 Security Breach | Security Officer |
| 💾 Data Loss | Backup Admin |
| 👥 User Issues | Help Desk |

---

## 🎯 Performance Targets

| Metric | Target |
|--------|--------|
| API Response | < 100ms |
| AI Response (short) | 2-5 sec |
| AI Response (long) | 10-30 sec |
| Uptime | > 99.5% |
| Concurrent Users | 100+ |

---

## 🔢 Version Information

| Item | Version |
|------|---------|
| Platform | 1.0.0 |
| Node.js | 20 LTS |
| PostgreSQL | 15 |
| Redis | 7 |
| LLaMA | 3 8B Q4_K_M |
| Flutter | 3.0+ |

---

## 🆘 Critical Passwords

**⚠️ Store securely, never commit to git**

```
Admin Email: ___________________________
Admin Password: ________________________
Database Password: _____________________
Redis Password: ________________________
JWT Secret: ____________________________
Meilisearch Key: _______________________
```

---

## 📊 System Ports

| Port | Service | Access |
|------|---------|--------|
| 22 | SSH | Internal only |
| 80 | HTTP | Redirect to 443 |
| 443 | HTTPS | Public |
| 3000 | API | Internal (Docker) |
| 5432 | PostgreSQL | Internal (Docker) |
| 6379 | Redis | Internal (Docker) |
| 7700 | Meilisearch | Internal (Docker) |
| 8080 | LLaMA | Internal (Docker) |

---

## 🎓 User Training Path

### Week 1: Basics (All Users)
- App navigation
- Send messages
- Use AI assistant

### Week 2: Advanced (All Users)
- Voice calls
- Calendar
- Profile settings

### Week 3: Best Practices (All Users)
- Security awareness
- Communication etiquette
- Productivity tips

### Admin Training (Managers Only)
- User management
- App version control
- System monitoring
- RAG document upload

---

## 📈 Success Metrics

- [ ] System uptime > 99.5%
- [ ] API response < 100ms
- [ ] User adoption > 95%
- [ ] Employee satisfaction > 80%
- [ ] AI query success > 90%

---

## ✅ Daily Admin Checklist

```
[ ] Run: bash scripts/monitor.sh
[ ] Check: docker-compose ps (all "Up")
[ ] Review: docker-compose logs (no errors)
[ ] Verify: df -h (disk space OK)
[ ] Confirm: Backup completed (check /backups)
```

---

## 🔄 Backup Information

**Location:** `/srv/Lone Star Chat/backups/`  
**Schedule:** Daily at 2:00 AM  
**Retention:** 30 days  
**Format:** tar.gz compressed  
**Contents:** Database, Redis, uploads, config

**Manual Backup:**
```bash
sudo bash /srv/Lone\ Star\ Chat/scripts/backup.sh
```

**Restore:**
```bash
sudo bash /srv/Lone\ Star\ Chat/scripts/restore.sh /path/to/backup.tar.gz
```

---

## 🎯 Quick Access Commands

Copy these to your `.bashrc` for easy access:

```bash
# Lone Star Chat Shortcuts
alias lsc-status='docker-compose -f /srv/Lone\ Star\ Chat/docker-compose.yml ps'
alias lsc-logs='docker-compose -f /srv/Lone\ Star\ Chat/docker-compose.yml logs -f'
alias lsc-backup='sudo bash /srv/Lone\ Star\ Chat/scripts/backup.sh'
alias lsc-monitor='sudo bash /srv/Lone\ Star\ Chat/scripts/monitor.sh'
alias lsc-restart='docker-compose -f /srv/Lone\ Star\ Chat/docker-compose.yml restart'
```

---

**Quick Reference Version:** 1.0.0  
**Last Updated:** 2025-01-04  
**Print Date:** __________

---

<div align="center">

**Keep this reference handy for daily operations**

**For detailed information, see full documentation in `/srv/Lone Star Chat/docs/`**

</div>

---

## 📝 Notes Section

```
_____________________________________________
_____________________________________________
_____________________________________________
_____________________________________________
_____________________________________________
_____________________________________________
```
