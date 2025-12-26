# 🚀 Migration Guide: V2 → V3 Lite (PostgreSQL + Redis)

## ✅ What We Created

### 1. Database Files
- `backend/dal/postgres-client.js` - PostgreSQL connection pool
- `backend/dal/redis-client.js` - Redis caching client
- `backend/scripts/create-schema.sql` - Database schema (tables + indexes)
- `backend/scripts/migrate-to-postgres.js` - Migration script (JSON → PostgreSQL)

### 2. Services
- `backend/services/message-service-v3.js` - Messages with Redis cache
- `backend/services/dm-service-v3.js` - Direct messages with cache

### 3. Configuration
- `backend/.env.v3` - Environment variables for PostgreSQL + Redis

---

## 📋 Prerequisites

Server already has:
- ✅ PostgreSQL 12 installed
- ✅ Database `lonestar` created
- ✅ User `lonestar_user` with password `lonestar2025`
- ✅ Schema SQL file uploaded to `/root/create-schema.sql`

Still need:
- 🔲 Redis installation
- 🔲 Apply database schema
- 🔲 Run migration script

---

## 🔧 Installation Steps

### Step 1: Install Redis on Server

```bash
ssh root@5.249.160.54

# Install Redis
apt install -y redis-server

# Configure password
nano /etc/redis/redis.conf
# Find and uncomment: requirepass lonestar2025

# Restart Redis
systemctl restart redis
systemctl enable redis

# Test connection
redis-cli
> AUTH lonestar2025
> PING
PONG
> EXIT
```

### Step 2: Apply Database Schema

```bash
# On server
ssh root@5.249.160.54

# Apply schema
sudo -u postgres psql -d lonestar -f /root/create-schema.sql

# Verify tables
sudo -u postgres psql -d lonestar -c "\dt"
# Should show: users, channels, messages, dm_messages, call_logs
```

### Step 3: Upload Backend Files

```bash
# On local machine
cd "/Users/svetanaborvinskaia/Desktop/Lone Star Chat"

# Upload new files
scp backend/dal/postgres-client.js root@5.249.160.54:/root/lonestar-chat/backend/dal/
scp backend/dal/redis-client.js root@5.249.160.54:/root/lonestar-chat/backend/dal/
scp backend/services/message-service-v3.js root@5.249.160.54:/root/lonestar-chat/backend/services/
scp backend/services/dm-service-v3.js root@5.249.160.54:/root/lonestar-chat/backend/services/
scp backend/scripts/migrate-to-postgres.js root@5.249.160.54:/root/lonestar-chat/backend/scripts/
scp backend/.env.v3 root@5.249.160.54:/root/lonestar-chat/backend/.env.v3
```

### Step 4: Run Migration

```bash
# On server
ssh root@5.249.160.54
cd /root/lonestar-chat/backend

# Install dependencies (if not already)
npm install pg redis

# Run migration
node scripts/migrate-to-postgres.js

# Expected output:
# 📤 Migrating users...
# ✅ Migrated 68 users
# 📤 Migrating channels...
# ✅ Migrated X channels
# 📤 Migrating messages...
# ✅ Migrated X messages
# 📤 Migrating DM messages...
# ✅ Migrated X DM messages
# 🔍 Verifying migration...
# 📊 Migration Summary:
#    Users: 68
#    Channels: X
#    Messages: X
#    DM Messages: X
# 🎉 Migration completed successfully!
```

### Step 5: Update Main Server File

You need to update `server-chat.js` to use new services:

```javascript
// At the top, replace old requires with:
const messageService = require('./services/message-service-v3');
const dmService = require('./services/dm-service-v3');

// Update message loading endpoint:
app.get('/api/messages/:channelId', async (req, res) => {
  try {
    const messages = await messageService.getMessages(req.params.channelId);
    res.json(messages);
  } catch (error) {
    console.error('Error loading messages:', error);
    res.status(500).json({ error: 'Failed to load messages' });
  }
});

// Update message sending via Socket.io:
socket.on('send-message', async (data) => {
  try {
    const message = await messageService.sendMessage(
      data.channelId,
      data.userId,
      data.userName,
      data.userAvatar,
      data.text,
      data.attachments || []
    );
    
    io.to(data.channelId).emit('new-message', message);
  } catch (error) {
    console.error('Error sending message:', error);
    socket.emit('message-error', { error: 'Failed to send message' });
  }
});

// Update DM endpoints similarly...
```

### Step 6: Restart Server

```bash
# On server
pm2 restart lonestar-chat

# Check logs
pm2 logs lonestar-chat

# Should see:
# ✅ PostgreSQL connected
# ✅ Redis connected
```

---

## 🧪 Testing

### Test 1: Message Load Speed

```bash
# Before (V2 with JSON files): ~500ms
# After (V3 with PostgreSQL + Redis): ~50-100ms (first request), ~0.2ms (cached)

curl -w "\nTime: %{time_total}s\n" http://5.249.160.54:666/api/messages/general
```

### Test 2: Search

```bash
# Full-text search (new feature!)
curl "http://5.249.160.54:666/api/messages/search?q=meeting"
```

### Test 3: Cache Hit Rate

Check Redis after a few requests:

```bash
redis-cli -a lonestar2025
> KEYS messages:*
> TTL messages:general:50:0
> GET messages:general:50:0
```

---

## 📊 Expected Performance Improvements

| Metric | V2 (JSON) | V3 Lite (PostgreSQL + Redis) |
|--------|-----------|------------------------------|
| Message Load | ~500ms | 50-100ms (first), 0.2ms (cached) |
| Search | ❌ Not available | ✅ <200ms full-text search |
| Max Users | ~100 | 1000+ |
| Concurrent Connections | Limited | High (connection pooling) |
| Data Size Limit | ~10MB practical | Unlimited |
| Query Flexibility | None | Full SQL + indexes |

---

## 🔄 Rollback Plan

If something goes wrong:

```bash
# 1. Stop new server
pm2 stop lonestar-chat

# 2. Restore old code
cd /root/lonestar-chat/backend
git checkout server-chat.js  # or restore from backup

# 3. Restart
pm2 restart lonestar-chat

# Your JSON files are untouched, so V2 will work immediately
```

---

## 🎯 Next Steps (Optional)

### Phase 2A: Add User Service with PostgreSQL
- Migrate user authentication to PostgreSQL
- Add user search functionality
- Cache user profiles in Redis

### Phase 2B: Add Analytics
- Track message counts per channel
- User activity metrics
- Popular channels

### Phase 3: AI RAG (when ready)
- Install Qdrant for vector embeddings
- Natural language queries: "find when Jean sent last message"
- Semantic search across all messages

### Phase 4: Scale to ScyllaDB (if needed)
- When you hit 1000+ concurrent users
- Migrate PostgreSQL → ScyllaDB
- Add Kafka/NATS for message queue

---

## 🆘 Troubleshooting

### PostgreSQL connection error
```bash
# Check PostgreSQL is running
systemctl status postgresql

# Check connection
sudo -u postgres psql -d lonestar -c "SELECT 1;"

# Grant permissions again if needed
sudo -u postgres psql -d lonestar -c "GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO lonestar_user;"
```

### Redis connection error
```bash
# Check Redis is running
systemctl status redis

# Test connection
redis-cli -a lonestar2025 PING

# Check password in config
grep "requirepass" /etc/redis/redis.conf
```

### Migration script errors
```bash
# Check file paths
ls -la backend/database/
ls -la channels-definition.json

# Run with debug
node scripts/migrate-to-postgres.js 2>&1 | tee migration.log
```

---

## ✅ Checklist

- [ ] Redis installed and running
- [ ] Database schema applied
- [ ] Migration script executed successfully
- [ ] Backend files uploaded
- [ ] Server code updated to use new services
- [ ] Server restarted
- [ ] Message loading tested (<100ms)
- [ ] Search tested
- [ ] Cache tested (Redis keys exist)
- [ ] DM messages working
- [ ] No errors in PM2 logs

---

## 📞 Current Status

**Created (Local):**
- ✅ All DAL and service files
- ✅ Migration script
- ✅ SQL schema
- ✅ This README

**On Server:**
- ✅ PostgreSQL installed
- ✅ Database and user created
- ✅ Schema SQL file uploaded
- ⏳ Waiting for server to stabilize to apply schema
- ⏳ Redis not yet installed

**Next Action:**
When server is stable, execute Steps 1-6 above.

---

**Ready to proceed when server load normalizes! 🚀**
