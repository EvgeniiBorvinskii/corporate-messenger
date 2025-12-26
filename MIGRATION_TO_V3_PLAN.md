# 🚀 MIGRATION PLAN: Version 2 → Version 3 (Enterprise)
**Target:** 3000+ employees corporate messenger  
**Timeline:** Phased migration (4-6 weeks)  
**Risk Level:** Medium (with proper testing)

---

## 🎯 Цели Version 3

### Performance Goals:
- ⚡ Message load time: < 100ms (vs current ~500ms)
- ⚡ Search query: < 200ms
- ⚡ File upload: < 1s for 10MB
- ⚡ Concurrent users: 3000+ online
- ⚡ Messages per second: 1000+

### New Features:
- 🔍 Full-text search across all messages
- 🤖 AI Chat with database query capabilities (RAG)
- 📊 Analytics dashboard
- 🔐 Advanced permissions system
- 📱 Better mobile performance
- 🌐 CDN for media files

---

## 🏗️ NEW ARCHITECTURE (Version 3)

```
┌─────────────────────────────────────────────────────────────┐
│                     Flutter Mobile App                       │
│                  (iOS + Android Ready)                       │
└────────────────────┬────────────────────────────────────────┘
                     │ HTTP/WebSocket
                     ▼
┌─────────────────────────────────────────────────────────────┐
│                   API Gateway / Load Balancer                │
│                    (Nginx or Traefik)                        │
└────┬────────────────────┬─────────────────────┬─────────────┘
     │                    │                     │
     ▼                    ▼                     ▼
┌─────────────┐  ┌─────────────────┐  ┌──────────────────┐
│  Node.js    │  │   Node.js       │  │   AI Service     │
│  API Server │  │ WebSocket Server│  │  (Llama + RAG)   │
│    (REST)   │  │   (Real-time)   │  │                  │
└──────┬──────┘  └────────┬────────┘  └────────┬─────────┘
       │                  │                     │
       ▼                  ▼                     ▼
┌──────────────────────────────────────────────────────────┐
│                      Redis Cluster                        │
│         (Cache: last messages, online users)              │
└──────┬───────────────────────────────────────────────────┘
       │
       ▼
┌──────────────────────────────────────────────────────────┐
│                    Kafka / NATS                           │
│          (Message Queue: delivery, sync)                  │
└──────┬───────────────────────────────────────────────────┘
       │
       ▼
┌──────────────────────────────────────────────────────────┐
│                    ScyllaDB Cluster                       │
│        (Primary Storage: messages, users, channels)       │
└──────────────────────────────────────────────────────────┘
       │
       ▼
┌──────────────────────────────────────────────────────────┐
│                  MinIO / S3 Storage                       │
│        (Files: images, videos, documents)                 │
└──────────────────────────────────────────────────────────┘
       │
       ▼
┌──────────────────────────────────────────────────────────┐
│                  Vector Database                          │
│            (Qdrant/Weaviate for AI RAG)                   │
└──────────────────────────────────────────────────────────┘
```

---

## 📅 PHASED MIGRATION PLAN

### **Phase 1: Infrastructure Setup (Week 1)**

#### Tasks:
1. **Install ScyllaDB on server 5.249.160.54**
   ```bash
   # ScyllaDB installation
   docker pull scylladb/scylla:latest
   docker run -d --name scylladb -p 9042:9042 scylladb/scylla
   ```

2. **Install Redis**
   ```bash
   docker pull redis:alpine
   docker run -d --name redis -p 6379:6379 redis:alpine
   ```

3. **Install MinIO (S3-compatible)**
   ```bash
   docker pull minio/minio
   docker run -d --name minio \
     -p 9000:9000 -p 9001:9001 \
     -e MINIO_ROOT_USER=admin \
     -e MINIO_ROOT_PASSWORD=adminpass \
     minio/minio server /data --console-address ":9001"
   ```

4. **Install NATS (lightweight alternative to Kafka)**
   ```bash
   docker pull nats:alpine
   docker run -d --name nats -p 4222:4222 -p 8222:8222 nats:alpine
   ```

5. **Install Vector DB (Qdrant for AI)**
   ```bash
   docker pull qdrant/qdrant
   docker run -d --name qdrant -p 6333:6333 qdrant/qdrant
   ```

#### Deliverables:
- [ ] All services running in Docker
- [ ] Health checks passing
- [ ] Monitoring setup (Prometheus + Grafana)

---

### **Phase 2: Database Migration (Week 2)**

#### 2.1 Design ScyllaDB Schema

**Messages Table:**
```cql
CREATE KEYSPACE lonestar WITH replication = {
  'class': 'SimpleStrategy', 
  'replication_factor': 1
};

CREATE TABLE lonestar.messages (
  channel_id text,
  message_id timeuuid,
  user_id text,
  user_name text,
  user_avatar text,
  text text,
  attachments list<text>,
  timestamp timestamp,
  likes set<text>,
  PRIMARY KEY (channel_id, message_id)
) WITH CLUSTERING ORDER BY (message_id DESC);

CREATE TABLE lonestar.users (
  user_id text PRIMARY KEY,
  email text,
  full_name text,
  department text,
  roles set<text>,
  avatar_url text,
  status text,
  last_seen timestamp
);

CREATE TABLE lonestar.dm_messages (
  conversation_id text,
  message_id timeuuid,
  sender_id text,
  receiver_id text,
  text text,
  attachments list<text>,
  timestamp timestamp,
  read boolean,
  PRIMARY KEY (conversation_id, message_id)
) WITH CLUSTERING ORDER BY (message_id DESC);
```

#### 2.2 Migrate Data from JSON to ScyllaDB

**Migration Script:**
```javascript
// backend/scripts/migrate-to-scylladb.js
const cassandra = require('cassandra-driver');
const fs = require('fs');

const client = new cassandra.Client({
  contactPoints: ['127.0.0.1'],
  localDataCenter: 'datacenter1',
  keyspace: 'lonestar'
});

async function migrateMessages() {
  const messages = JSON.parse(fs.readFileSync('./database/messages.json'));
  
  for (const [channelId, msgs] of Object.entries(messages)) {
    for (const msg of msgs) {
      await client.execute(
        `INSERT INTO messages (channel_id, message_id, user_id, user_name, 
         user_avatar, text, attachments, timestamp) 
         VALUES (?, now(), ?, ?, ?, ?, ?, ?)`,
        [channelId, msg.userId, msg.userName, msg.userAvatar, 
         msg.text, msg.attachments, new Date(msg.timestamp)]
      );
    }
  }
  console.log('✅ Messages migrated to ScyllaDB');
}

async function migrateUsers() {
  const users = JSON.parse(fs.readFileSync('./database/users.json'));
  
  for (const [userId, user] of Object.entries(users)) {
    await client.execute(
      `INSERT INTO users (user_id, email, full_name, department, 
       roles, avatar_url, status) 
       VALUES (?, ?, ?, ?, ?, ?, ?)`,
      [userId, user.email, user.full_name, user.department,
       user.roles, user.avatar_url, user.status]
    );
  }
  console.log('✅ Users migrated to ScyllaDB');
}

migrateMessages()
  .then(() => migrateUsers())
  .then(() => client.shutdown());
```

#### Deliverables:
- [ ] ScyllaDB schema created
- [ ] All data migrated (68 users, 20+ messages)
- [ ] Validation: data integrity check

---

### **Phase 3: Backend Refactoring (Week 3)**

#### 3.1 Create Data Access Layer (DAL)

**Structure:**
```
backend/
  dal/
    scylla-client.js     - ScyllaDB connection
    redis-client.js      - Redis connection
    nats-client.js       - NATS connection
    minio-client.js      - MinIO client
    
  services/
    message-service.js   - Message CRUD
    user-service.js      - User management
    cache-service.js     - Redis operations
    file-service.js      - MinIO uploads
    
  api/
    messages-api.js      - REST endpoints
    users-api.js
    channels-api.js
```

#### 3.2 Implement Redis Caching

**Example:**
```javascript
// backend/services/cache-service.js
const redis = require('redis');
const client = redis.createClient({ url: 'redis://localhost:6379' });

async function getCachedMessages(channelId) {
  const cached = await client.get(`messages:${channelId}:latest`);
  if (cached) return JSON.parse(cached);
  
  // Fetch from ScyllaDB
  const messages = await getMessagesFromDB(channelId);
  
  // Cache for 5 minutes
  await client.setEx(`messages:${channelId}:latest`, 300, JSON.stringify(messages));
  
  return messages;
}
```

#### Deliverables:
- [ ] DAL layer complete
- [ ] Redis caching for hot data
- [ ] NATS pub/sub for real-time

---

### **Phase 4: AI RAG Implementation (Week 4)**

#### 4.1 Setup Vector Database (Qdrant)

**Embed all messages:**
```javascript
// backend/services/ai-rag-service.js
const { QdrantClient } = require('@qdrant/js-client-rest');
const { OpenAI } = require('openai');

const qdrant = new QdrantClient({ url: 'http://localhost:6333' });
const openai = new OpenAI({ apiKey: process.env.OPENAI_API_KEY });

async function embedMessage(message) {
  const embedding = await openai.embeddings.create({
    model: 'text-embedding-ada-002',
    input: message.text
  });
  
  await qdrant.upsert('messages', {
    points: [{
      id: message.message_id,
      vector: embedding.data[0].embedding,
      payload: {
        channel_id: message.channel_id,
        user_name: message.user_name,
        text: message.text,
        timestamp: message.timestamp
      }
    }]
  });
}
```

#### 4.2 Natural Language to Database Query

**AI Agent:**
```javascript
// backend/services/ai-query-agent.js
const { ChatOpenAI } = require('langchain/chat_models/openai');
const { SqlDatabase } = require('langchain/sql_db');

const llm = new ChatOpenAI({ temperature: 0 });
const db = new SqlDatabase({ /* ScyllaDB connection */ });

async function processNaturalLanguageQuery(userQuery) {
  // Example: "can you find when Jean sent last message"
  
  const prompt = `
  Convert this natural language query to CQL:
  "${userQuery}"
  
  Database schema:
  - messages(channel_id, message_id, user_id, user_name, text, timestamp)
  
  Generate only the CQL query.
  `;
  
  const response = await llm.call([{ role: 'user', content: prompt }]);
  const cqlQuery = response.content;
  
  // Execute query
  const result = await client.execute(cqlQuery);
  
  return formatResult(result);
}
```

#### Examples:

**Query 1:** "can you find when Jean send last message"
```cql
SELECT timestamp, text FROM lonestar.messages 
WHERE user_name = 'Jean' 
ORDER BY message_id DESC 
LIMIT 1;
```

**Query 2:** "show me all messages between 2 and 3 pm"
```cql
SELECT * FROM lonestar.messages 
WHERE timestamp >= '2025-12-19 14:00:00' 
  AND timestamp < '2025-12-19 15:00:00';
```

#### Deliverables:
- [ ] Qdrant vector DB with all messages embedded
- [ ] AI agent for natural language queries
- [ ] RAG search working

---

### **Phase 5: Frontend Updates (Week 5)**

#### 5.1 Update API Clients

**New services:**
```dart
// mobile/lib/services/scylla_message_service.dart
class ScyllaMessageService {
  static const baseUrl = 'http://5.249.160.54:666/api/v3';
  
  Future<List<Message>> getMessages(String channelId, {int limit = 50}) async {
    final response = await http.get('$baseUrl/channels/$channelId/messages?limit=$limit');
    return (response.data as List).map((m) => Message.fromJson(m)).toList();
  }
  
  Future<void> sendMessage(String channelId, String text) async {
    await http.post('$baseUrl/channels/$channelId/messages', data: {
      'text': text,
      'timestamp': DateTime.now().toIso8601String()
    });
  }
}
```

#### 5.2 AI Chat UI

```dart
// mobile/lib/screens/ai_chat_screen.dart
class AIChatScreen extends StatelessWidget {
  final AIService aiService = AIService();
  
  Widget build(BuildContext context) {
    return Scaffold(
      body: Column(
        children: [
          ChatMessagesList(),
          AIInputField(
            onSubmit: (query) async {
              // Natural language query
              final result = await aiService.queryDatabase(query);
              displayResult(result);
            }
          ),
          // Quick actions
          Row(
            children: [
              ActionChip(label: 'Find messages'),
              ActionChip(label: 'Search by user'),
              ActionChip(label: 'Time range'),
            ]
          )
        ]
      )
    );
  }
}
```

#### Deliverables:
- [ ] API clients updated
- [ ] AI Chat UI
- [ ] Performance optimizations

---

### **Phase 6: Testing & Deployment (Week 6)**

#### 6.1 Load Testing

**Test scenarios:**
- 1000 concurrent users
- 100 messages/second
- Search queries under load
- File uploads (10MB+)

**Tools:**
- k6 (load testing)
- Artillery (WebSocket stress test)

#### 6.2 Rollout Plan

**Step 1:** Parallel run (V2 + V3 both active)
**Step 2:** Gradual migration (10% → 50% → 100%)
**Step 3:** Monitor metrics
**Step 4:** Full cutover

#### Deliverables:
- [ ] Load tests passing
- [ ] Rollback plan ready
- [ ] Production deployment

---

## 🔧 REQUIRED PACKAGES

### Backend (Node.js):
```json
{
  "dependencies": {
    "cassandra-driver": "^4.7.2",
    "redis": "^4.6.0",
    "nats": "^2.15.0",
    "minio": "^7.1.0",
    "@qdrant/js-client-rest": "^1.7.0",
    "langchain": "^0.1.0",
    "openai": "^4.20.0"
  }
}
```

### Frontend (Flutter):
No major changes needed - same API structure.

---

## 📊 PERFORMANCE COMPARISON

| Metric | Version 2 | Version 3 (Target) |
|--------|-----------|-------------------|
| Message Load | 500ms | < 100ms |
| Search Query | N/A | < 200ms |
| Max Users | 100 | 3000+ |
| Storage | JSON (500KB) | ScyllaDB (unlimited) |
| Caching | None | Redis (0.2ms) |
| AI Search | None | Full RAG |

---

## 💰 COST ESTIMATE

**Current:** $20/month (single VPS)

**Version 3:**
- VPS (8GB RAM, 4 CPU): $40/month
- OR dedicated server: $80-150/month
- Storage (MinIO): included
- No external services (all self-hosted)

**Total:** ~$40-150/month (depends on load)

---

## ⚠️ RISKS & MITIGATION

### Risk 1: Data loss during migration
**Mitigation:** Keep V2 running, V3 in parallel

### Risk 2: Performance issues
**Mitigation:** Load testing before cutover

### Risk 3: AI costs (OpenAI API)
**Mitigation:** Use local Llama for embeddings

---

## ✅ GO/NO-GO CHECKLIST

Before starting migration:
- [ ] Version 2 backup complete
- [ ] Server resources sufficient (8GB+ RAM)
- [ ] Team trained on new architecture
- [ ] Rollback plan documented
- [ ] Monitoring setup ready

---

## 📞 NEXT STEPS

1. **Review this plan** - any questions?
2. **Provision infrastructure** - Docker containers
3. **Start Phase 1** - Infrastructure setup
4. **Weekly check-ins** - track progress

**Ready to start?** 🚀
