# Backend Updates Required for New Features

## 📋 Обзор изменений

Для работы новых функций нужно добавить следующие API endpoints на backend:

---

## 🗑️ 1. Удаление сообщений

### DELETE `/api/team-chat/:teamRole/messages/:messageId`
**Описание:** Удалить сообщение из командного чата

**Параметры:**
- `teamRole` - роль команды (sales, support и т.д.)
- `messageId` - ID сообщения

**Логика:**
```javascript
// Проверить права:
// - Пользователь может удалить только свои сообщения
// - MASTER может удалить любые сообщения
if (message.userId !== req.user.id && !req.user.roles.includes('MASTER')) {
  return res.status(403).json({ error: 'No permission' });
}

await Message.findByIdAndDelete(messageId);
```

### DELETE `/api/messages/dm/:userId/messages/:messageId`
**Описание:** Удалить сообщение из личного чата

**Параметры:**
- `userId` - ID собеседника
- `messageId` - ID сообщения

**Логика:** Аналогично командным чатам

---

## 📞 2. WebRTC Звонки (Socket.io)

### Добавить Socket.io обработчики в `server-chat-current.js`:

```javascript
const io = require('socket.io')(server, {
  cors: {
    origin: "*",
    methods: ["GET", "POST"]
  }
});

// Хранилище активных звонков
const activeCalls = new Map();

io.on('connection', (socket) => {
  console.log('🔌 Client connected:', socket.id);

  // Аутентификация
  socket.on('authenticate', (token) => {
    try {
      const user = verifyToken(token);
      socket.userId = user.id;
      socket.join(`user_${user.id}`);
    } catch (err) {
      socket.disconnect();
    }
  });

  // 📞 Начать звонок
  socket.on('start_call', async (data) => {
    const { to, isVideo, sdp } = data;
    const callId = `call_${Date.now()}_${socket.userId}_${to}`;
    
    activeCalls.set(callId, {
      from: socket.userId,
      to,
      isVideo,
      startTime: new Date(),
    });

    // Уведомить получателя
    io.to(`user_${to}`).emit('incoming_call', {
      callId,
      from: socket.userId,
      isVideo,
      sdp,
    });

    socket.callId = callId;
  });

  // ✅ Принять звонок
  socket.on('accept_call', async (data) => {
    const { callId } = data;
    const call = activeCalls.get(callId);
    
    if (call) {
      io.to(`user_${call.from}`).emit('call_accepted', {
        callId,
        userId: socket.userId,
      });
    }
  });

  // ❌ Отклонить звонок
  socket.on('reject_call', (data) => {
    const { callId } = data;
    const call = activeCalls.get(callId);
    
    if (call) {
      io.to(`user_${call.from}`).emit('call_rejected', { callId });
      activeCalls.delete(callId);
    }
  });

  // 📞 Завершить звонок
  socket.on('end_call', async (data) => {
    const { callId } = data;
    const call = activeCalls.get(callId);
    
    if (call) {
      const duration = Math.floor((new Date() - call.startTime) / 1000);
      
      // Сохранить запись о звонке в DM чат
      await Message.create({
        chatId: `dm_${call.from}_${call.to}`,
        userId: call.from,
        content: `${call.isVideo ? '📹' : '📞'} Call - ${formatDuration(duration)}`,
        type: call.isVideo ? 'video_call' : 'voice_call',
        metadata: {
          duration,
          from: call.from,
          to: call.to,
          endTime: new Date(),
        }
      });

      // Уведомить обоих
      io.to(`user_${call.from}`).emit('call_ended', { callId, duration });
      io.to(`user_${call.to}`).emit('call_ended', { callId, duration });
      
      activeCalls.delete(callId);
    }
  });

  // 🧊 ICE Candidate
  socket.on('ice_candidate', (data) => {
    const { callId, candidate, sdpMid, sdpMLineIndex } = data;
    const call = activeCalls.get(callId);
    
    if (call) {
      const targetUser = call.from === socket.userId ? call.to : call.from;
      io.to(`user_${targetUser}`).emit('ice_candidate', {
        callId,
        candidate,
        sdpMid,
        sdpMLineIndex,
      });
    }
  });

  // 📡 SDP Offer
  socket.on('sdp_offer', (data) => {
    const { callId, sdp } = data;
    const call = activeCalls.get(callId);
    
    if (call) {
      io.to(`user_${call.to}`).emit('sdp_offer', {
        callId,
        sdp,
      });
    }
  });

  // 📡 SDP Answer
  socket.on('sdp_answer', (data) => {
    const { callId, sdp } = data;
    const call = activeCalls.get(callId);
    
    if (call) {
      io.to(`user_${call.from}`).emit('sdp_answer', {
        callId,
        sdp,
      });
    }
  });

  socket.on('disconnect', () => {
    console.log('🔌 Client disconnected:', socket.id);
    
    // Завершить активный звонок при отключении
    if (socket.callId) {
      socket.emit('end_call', { callId: socket.callId });
    }
  });
});

function formatDuration(seconds) {
  const h = Math.floor(seconds / 3600);
  const m = Math.floor((seconds % 3600) / 60);
  const s = seconds % 60;
  
  if (h > 0) return `${h}:${String(m).padStart(2, '0')}:${String(s).padStart(2, '0')}`;
  return `${m}:${String(s).padStart(2, '0')}`;
}
```

---

## 📦 Зависимости

Добавить в `package.json` (если еще нет):

```json
{
  "dependencies": {
    "socket.io": "^4.6.0"
  }
}
```

---

## 🚀 Установка

```bash
cd /root/lonestar-chat/backend
npm install socket.io
pm2 restart lonestar-chat
```

---

## 📝 Модель сообщения о звонке

Обновить схему Message для поддержки типов звонков:

```javascript
type: {
  type: String,
  enum: ['text', 'image', 'file', 'voice_call', 'video_call'],
  default: 'text'
},
metadata: {
  type: Object,
  default: {}
}
```

---

## 🔒 Безопасность

1. **Аутентификация:** Проверять JWT token при подключении к Socket.io
2. **Авторизация:** Проверять что пользователь может звонить этому контакту
3. **Rate limiting:** Ограничить количество звонков в минуту
4. **Валидация:** Проверять все входящие данные

---

## 📊 Тестирование

### Тест удаления сообщения:
```bash
curl -X DELETE http://5.249.160.54:666/api/team-chat/sales/messages/MSG_ID \
  -H "Authorization: Bearer YOUR_TOKEN"
```

### Тест WebRTC:
1. Открыть два клиента
2. Инициировать звонок от первого
3. Принять на втором
4. Проверить аудио/видео
5. Завершить звонок
6. Проверить что запись о звонке появилась в DM чате

---

## 🎯 Приоритет

1. **Высокий:** Удаление сообщений (просто REST API)
2. **Высокий:** WebRTC Socket.io handlers (требует больше работы)
3. **Средний:** История звонков в чатах (автоматически при завершении звонка)

---

## 📝 Примечания

- Все WebRTC signaling идет через Socket.io
- Сам медиа-трафик идет напрямую между клиентами (P2P)
- STUN сервер: `stun:stun.l.google.com:19302` (используется в мобильном приложении)
- Для production рекомендуется добавить TURN сервер для работы за NAT

---

Дата создания: 10 декабря 2025
