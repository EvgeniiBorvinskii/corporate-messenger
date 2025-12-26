# 🎯 REVISED PLAN: Lightweight Migration (Version 3 Lite)

## ⚠️ ПРОБЛЕМА
Сервер 5.249.160.54 перегружен (Load Average: 7.41):
- Docker Scy llaDB требует много ресурсов
- Запуск 5 контейнеров одновременно перегружает систему
- Сервер используется для других проектов (YouTube backend, Gupics, etc.)

## ✅ РЕШЕНИЕ: Поэтапная миграция

### PHASE 1 LITE: PostgreSQL + Redis (Легкая версия)
**Timeline:** 1-2 weeks

Вместо:
❌ ScyllaDB (тяжелый)
❌ Kafka/NATS (не критично сейчас)
❌ MinIO (можно позже)
❌ Qdrant (AI - позже)

Используем:
✅ PostgreSQL 15 (легкий, быстрый, знакомый)
✅ Redis 7 (кэш)
✅ Улучшенный Node.js backend

**Преимущества:**
- Работает на текущем сервере
- Быстрая миграция (1-2 дня)
- Сразу видно улучшение производительности
- Легко откатиться

**Производительность:**
- Message load: 500ms → 50-100ms ⚡
- Max users: 100 → 1000 
- Search: None → Full-text search ✅
- Redis cache: 0.2ms ✅

---

## 📊 СРАВНЕНИЕ ПЛАНОВ

| Feature | Version 2 | V3 Lite (PostgreSQL) | V3 Full (ScyllaDB) |
|---------|-----------|---------------------|-------------------|
| Database | JSON files | PostgreSQL | ScyllaDB |
| Cache | None | Redis | Redis |
| Queue | None | Optional | NATS/Kafka |
| File Storage | Local | Local | MinIO |
| AI Vector DB | None | Phase 2 | Qdrant |
| Max Users | 100 | 1000 | 3000+ |
| Setup Time | - | 1-2 days | 1-2 weeks |
| Server Load | Low | Medium | High |
| Migration Risk | - | Low | Medium |

---

## 🚀 PHASE 1 LITE: PostgreSQL Migration

### Step 1: Install PostgreSQL

```bash
ssh root@5.249.160.54

# Install PostgreSQL
apt update
apt install -y postgresql postgresql-contrib

# Start service
systemctl start postgresql
systemctl enable postgresql

# Create database
sudo -u postgres psql
```

```sql
CREATE DATABASE lonestar;
CREATE USER lonestar_user WITH ENCRYPTED PASSWORD 'lonestar2025';
GRANT ALL PRIVILEGES ON DATABASE lonestar TO lonestar_user;
\q
```

### Step 2: Create Schema

```sql
-- Connect to database
\c lonestar

-- Users table
CREATE TABLE users (
  id VARCHAR(50) PRIMARY KEY,
  email VARCHAR(255) UNIQUE NOT NULL,
  full_name VARCHAR(255),
  department VARCHAR(100),
  roles TEXT[],
  avatar_url TEXT,
  status VARCHAR(20) DEFAULT 'offline',
  last_seen TIMESTAMP,
  created_at TIMESTAMP DEFAULT NOW()
);

-- Channels table
CREATE TABLE channels (
  id VARCHAR(50) PRIMARY KEY,
  name VARCHAR(255) NOT NULL,
  description TEXT,
  type VARCHAR(20),
  category VARCHAR(50),
  allowed_roles TEXT[],
  is_push_enabled BOOLEAN DEFAULT false,
  created_at TIMESTAMP DEFAULT NOW()
);

-- Messages table (with indexes for fast queries)
CREATE TABLE messages (
  id VARCHAR(100) PRIMARY KEY,
  channel_id VARCHAR(50) REFERENCES channels(id),
  user_id VARCHAR(50) REFERENCES users(id),
  user_name VARCHAR(255),
  user_avatar TEXT,
  text TEXT NOT NULL,
  attachments JSONB DEFAULT '[]',
  timestamp TIMESTAMP DEFAULT NOW(),
  likes TEXT[] DEFAULT '{}'
);

-- Indexes for performance
CREATE INDEX idx_messages_channel ON messages(channel_id, timestamp DESC);
CREATE INDEX idx_messages_user ON messages(user_id);
CREATE INDEX idx_messages_timestamp ON messages(timestamp DESC);
CREATE INDEX idx_messages_text ON messages USING gin(to_tsvector('english', text));

-- DM Messages
CREATE TABLE dm_messages (
  id VARCHAR(100) PRIMARY KEY,
  conversation_id VARCHAR(100) NOT NULL,
  sender_id VARCHAR(50) REFERENCES users(id),
  receiver_id VARCHAR(50) REFERENCES users(id),
  text TEXT NOT NULL,
  attachments JSONB DEFAULT '[]',
  timestamp TIMESTAMP DEFAULT NOW(),
  read BOOLEAN DEFAULT false
);

CREATE INDEX idx_dm_conversation ON dm_messages(conversation_id, timestamp DESC);
CREATE INDEX idx_dm_unread ON dm_messages(receiver_id, read) WHERE read = false;
```

### Step 3: Install Redis (Simple)

```bash
# Install Redis
apt install -y redis-server

# Configure
nano /etc/redis/redis.conf
# Add: requirepass lonestar2025

# Restart
systemctl restart redis
systemctl enable redis

# Test
redis-cli
> AUTH lonestar2025
> PING
PONG
```

### Step 4: Migrate Data (JSON → PostgreSQL)

```javascript
// backend/scripts/migrate-to-postgres.js
const { Client } = require('pg');
const fs = require('fs');

const client = new Client({
  host: 'localhost',
  database: 'lonestar',
  user: 'lonestar_user',
  password: 'lonestar2025'
});

async function migrate() {
  await client.connect();
  
  // Migrate users
  const users = JSON.parse(fs.readFileSync('./database/users.json'));
  for (const [userId, user] of Object.entries(users)) {
    await client.query(`
      INSERT INTO users (id, email, full_name, department, roles, avatar_url, status)
      VALUES ($1, $2, $3, $4, $5, $6, $7)
      ON CONFLICT (id) DO UPDATE SET
        email = EXCLUDED.email,
        full_name = EXCLUDED.full_name
    `, [userId, user.email, user.full_name, user.department, 
        user.roles, user.avatar_url, user.status]);
  }
  console.log('✅ Users migrated');
  
  // Migrate channels
  const channels = JSON.parse(fs.readFileSync('../channels-definition.json'));
  for (const [channelId, channel] of Object.entries(channels)) {
    await client.query(`
      INSERT INTO channels (id, name, description, type, category, allowed_roles)
      VALUES ($1, $2, $3, $4, $5, $6)
      ON CONFLICT (id) DO NOTHING
    `, [channelId, channel.name, channel.description, channel.type,
        channel.category, channel.allowedRoles]);
  }
  console.log('✅ Channels migrated');
  
  // Migrate messages
  const messages = JSON.parse(fs.readFileSync('./database/messages.json'));
  for (const [channelId, msgs] of Object.entries(messages)) {
    for (const msg of msgs) {
      await client.query(`
        INSERT INTO messages (id, channel_id, user_id, user_name, user_avatar, text, attachments, timestamp)
        VALUES ($1, $2, $3, $4, $5, $6, $7, $8)
        ON CONFLICT (id) DO NOTHING
      `, [msg.id, channelId, msg.userId, msg.userName, msg.userAvatar,
          msg.text, JSON.stringify(msg.attachments), msg.timestamp]);
    }
  }
  console.log('✅ Messages migrated');
  
  await client.end();
  console.log('🎉 Migration complete!');
}

migrate().catch(console.error);
```

### Step 5: Update Backend

```javascript
// backend/dal/postgres-client.js
const { Pool } = require('pg');

const pool = new Pool({
  host: 'localhost',
  database: 'lonestar',
  user: 'lonestar_user',
  password: 'lonestar2025',
  max: 20,
  idleTimeoutMillis: 30000,
  connectionTimeoutMillis: 2000,
});

module.exports = pool;
```

```javascript
// backend/services/message-service.js
const pool = require('../dal/postgres-client');
const redis = require('../dal/redis-client');

async function getMessages(channelId, limit = 50) {
  // Try cache first
  const cacheKey = `messages:${channelId}:latest`;
  const cached = await redis.get(cacheKey);
  if (cached) return JSON.parse(cached);
  
  // Query PostgreSQL
  const result = await pool.query(`
    SELECT * FROM messages 
    WHERE channel_id = $1 
    ORDER BY timestamp DESC 
    LIMIT $2
  `, [channelId, limit]);
  
  const messages = result.rows;
  
  // Cache for 5 minutes
  await redis.setEx(cacheKey, 300, JSON.stringify(messages));
  
  return messages;
}

async function sendMessage(channelId, userId, text) {
  const messageId = `msg_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`;
  
  await pool.query(`
    INSERT INTO messages (id, channel_id, user_id, text)
    VALUES ($1, $2, $3, $4)
  `, [messageId, channelId, userId, text]);
  
  // Invalidate cache
  await redis.del(`messages:${channelId}:latest`);
  
  return messageId;
}

async function searchMessages(query) {
  const result = await pool.query(`
    SELECT *, ts_rank(to_tsvector('english', text), plainto_tsquery('english', $1)) as rank
    FROM messages
    WHERE to_tsvector('english', text) @@ plainto_tsquery('english', $1)
    ORDER BY rank DESC, timestamp DESC
    LIMIT 50
  `, [query]);
  
  return result.rows;
}

module.exports = { getMessages, sendMessage, searchMessages };
```

---

## 📈 EXPECTED IMPROVEMENTS

### Before (V2):
```
- Message load: 500ms (read JSON file)
- Search: Not available
- Max concurrent users: ~100
- Database size: 500KB (limited)
```

### After (V3 Lite):
```
- Message load: 50-100ms (PostgreSQL + Redis cache)
- Search: Full-text search < 200ms
- Max concurrent users: 1000+
- Database size: Unlimited (PostgreSQL)
- Redis cache: 0.2ms for hot data
```

---

## 🔄 ROLLBACK PLAN

If something goes wrong:

```bash
# 1. Stop new backend
pm2 stop lonestar-chat-v3

# 2. Restart V2
pm2 restart lonestar-chat

# 3. Restore from Version 2 backup
cd "Lone Star Chat Version 2 - 20251219/SERVER_BACKUP"
tar -xzf database-backup.tar.gz
scp -r database/ root@5.249.160.54:/root/lonestar-chat/backend/
```

---

## 🎯 NEXT STEPS (After V3 Lite is stable)

### Phase 2: Add AI RAG (2-3 weeks later)
- Install Qdrant for vector embeddings
- Implement natural language queries
- "can you find when Jean sent last message" ✅

### Phase 3: Scale to ScyllaDB (if needed for 3000+ users)
- Migrate PostgreSQL → ScyllaDB
- Add Kafka/NATS
- Add MinIO for files
- Full enterprise architecture

---

## ✅ DECISION

**Start with V3 Lite (PostgreSQL + Redis)?**

Advantages:
- ✅ Works on current server
- ✅ Quick migration (1-2 days)
- ✅ Immediate performance boost
- ✅ Low risk
- ✅ Easy rollback

Or wait and upgrade server first for full V3 (ScyllaDB)?

**Your choice?** 🤔
