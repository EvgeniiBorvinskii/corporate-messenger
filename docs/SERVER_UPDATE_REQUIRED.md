# 🚨 КРИТИЧЕСКАЯ ПРОБЛЕМА: Код не обновлён на сервере!

## ❌ Проблемы обнаружены:

### 1. NSMicrophoneUsageDescription - ИСПРАВЛЕНО ✅
Добавлено в `mobile/ios/Runner/Info.plist`:
```xml
<key>NSMicrophoneUsageDescription</key>
<string>Lone Star Chat needs access to your microphone for voice and video calls with team members</string>
<key>NSCameraUsageDescription</key>
<string>Lone Star Chat needs access to your camera for video calls with team members</string>
```

### 2. WebSocket 404 на сервере - НЕ ИСПРАВЛЕНО ❌

**Ошибка из логов**:
```
❌ WebSocket error: Connection to 'http://5.249.160.54:666/socket.io/?EIO=4&transport=websocket#' was not upgraded to websocket, HTTP status code: 404
```

**Причина**: На выделенном сервере `5.249.160.54:666` **НЕТ** обновлённого кода!

## 🚀 ЧТО НУЖНО СДЕЛАТЬ:

### Вариант А: Обновить код на выделенном сервере

1. **Подключиться к серверу**:
```bash
ssh user@5.249.160.54
```

2. **Найти папку с backend**:
```bash
# Возможные пути:
cd /var/www/chat-backend
# или
cd /home/user/chat-backend
# или
cd /opt/chat-backend
```

3. **Остановить текущий сервер**:
```bash
# Если используется PM2
pm2 stop chat-server
# или
pm2 stop all

# Если используется systemd
sudo systemctl stop chat-server

# Если просто node
pkill -f "node.*server-chat.js"
```

4. **Обновить файл `server-chat.js`**:
```bash
# Сделать backup
cp server-chat.js server-chat.js.backup

# Загрузить новый файл (через scp с вашего Mac)
```

5. **С вашего Mac загрузить новый файл**:
```bash
scp /Users/svetanaborvinskaia/Desktop/Lone\ Star\ Chat/backend/server-chat.js user@5.249.160.54:/path/to/backend/
```

6. **Перезапустить сервер**:
```bash
# Если PM2
pm2 start server-chat.js --name chat-server
pm2 save

# Если systemd
sudo systemctl start chat-server

# Если вручную
node server-chat.js > server.log 2>&1 &
```

7. **Проверить что сервер работает**:
```bash
curl http://localhost:666/
# Должен вернуть HTML страницу

# Проверить Socket.io
curl http://localhost:666/socket.io/
# НЕ должен возвращать 404
```

### Вариант Б: Использовать локальный сервер для разработки

Если нет доступа к выделенному серверу:

1. **Запустить локальный backend**:
```bash
cd /Users/svetanaborvinskaia/Desktop/Lone\ Star\ Chat/backend
node server-chat.js
```

2. **Изменить IP в приложении**:
```dart
// В mobile/lib/core/config/api_config.dart
static const List<String> suggestedIPs = [
  '192.168.51.238',  // ✅ Ваш локальный IP
  '5.249.160.54',    // Выделенный сервер
];
```

3. **Узнать свой локальный IP**:
```bash
ipconfig getifaddr en0  # WiFi
# или
ipconfig getifaddr en1  # Ethernet
```

4. **В приложении**: Profile → Settings → IP Address → выбрать локальный IP

## 📝 Что обновлено в server-chat.js:

### Добавлены Socket.io события для звонков:

```javascript
// Входящий звонок
socket.on('call:incoming', (data) => { ... });

// Звонок принят
socket.on('call:accepted', (data) => { ... });

// Звонок отклонён
socket.on('call:rejected', (data) => { ... });

// WebRTC SDP Offer
socket.on('call:offer', (data) => { ... });

// WebRTC SDP Answer
socket.on('call:answer', (data) => { ... });

// ICE Candidate
socket.on('call:ice-candidate', (data) => { ... });

// Завершение звонка
socket.on('call:end', (data) => { ... });

// Управление аудио/видео
socket.on('call:toggle-audio', (data) => { ... });
socket.on('call:toggle-video', (data) => { ... });
```

### Добавлены API endpoints:

```javascript
// История звонков
GET /api/calls/history
GET /api/calls/history/:otherUserId
DELETE /api/calls/history/:callId
```

## ✅ После обновления сервера:

1. **Пересобрать приложение**:
```bash
cd mobile
flutter clean
flutter pub get
flutter build ios --release --no-codesign
```

2. **Запустить на устройстве**:
```bash
flutter run --release
```

3. **Тестировать звонки**:
- Логи не должны показывать "WebSocket 404"
- Звонки не должны крашить приложение
- Должен быть доступ к микрофону и камере

## 🔍 Проверка что сервер обновлён:

```bash
# С Mac или с сервера
curl http://5.249.160.54:666/socket.io/ -v

# Ответ НЕ должен быть 404!
# Должно быть что-то типа:
# {"code":0,"message":"Transport unknown"}
# или
# 0{"sid":"...","upgrades":["websocket"],"pingInterval":25000,"pingTimeout":20000}
```

## ❗ ВАЖНО:

**БЕЗ ОБНОВЛЕНИЯ КОДА НА СЕРВЕРЕ ЗВОНКИ НЕ БУДУТ РАБОТАТЬ!**

Все изменения сделаны только в локальных файлах:
- ✅ `mobile/lib/services/call_service.dart` - обновлён
- ✅ `mobile/ios/Runner/Info.plist` - обновлён  
- ❌ `backend/server-chat.js` на сервере `5.249.160.54` - **НЕ ОБНОВЛЁН!**

---

**Дата**: 12 декабря 2025  
**Статус**: ⚠️ Требуется обновление сервера
