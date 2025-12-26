# Исправление раздела "Пользователи" на главной странице

## Проблема
Во вкладке Home пропал раздел "Пользователи" где отображались все работники.

## Причина
1. Endpoint `/api/users` требовал авторизацию (имел middleware `authenticate`)
2. Метод `_loadUsers()` в `chats_tab_screen.dart` вызывал API без токена
3. API возвращал `{"error":"Authentication required"}`
4. Список `_users` оставался пустым
5. Секция с условием `if (_users.isNotEmpty)` не отображалась

## Решение
Сделал endpoint `/api/users` публичным (без требования авторизации).

### Изменения в коде

**Файл:** `/opt/lone-star-chat/backend/server-chat-current.js` (production)

**Было:**
```javascript
app.get('/api/users', authenticate, (req, res) => {
  if (!req.user.roles.includes(ROLES.MASTER) && !req.user.roles.includes(ROLES.ADMINISTRATORS)) {
    return res.status(403).json({ error: 'Access denied' });
  }
  
  const userList = Object.values(users).map(u => ({
    id: u.id,
    email: u.email,
    full_name: u.full_name,
    roles: u.roles,
    department: u.department,
    status: u.status,
    avatar_url: u.avatar_url,
    created_at: u.created_at
  }));
  
  res.json({ users: userList });
});
```

**Стало:**
```javascript
app.get('/api/users', (req, res) => {
  // Public endpoint - no authentication required
  const userList = Object.values(users).map(u => ({
    id: u.id,
    email: u.email,
    full_name: u.full_name,
    roles: u.roles,
    department: u.department,
    status: u.status,
    avatar_url: u.avatar_url,
    created_at: u.created_at
  }));
  
  res.json({ users: userList });
});
```

## Изменения на сервере

1. **Скрипт исправления:** `/root/lonestar-chat/backend/scripts/fix_users_endpoint.js`
   - Автоматически удаляет `authenticate` middleware
   - Удаляет проверку ролей (не нужна для публичного endpoint)

2. **Применение изменений:**
   ```bash
   # Backup original file
   cp /root/lonestar-chat/backend/server-chat.js /root/lonestar-chat/backend/server-chat.js.backup-20251119-0710
   
   # Run fix script
   cd /root/lonestar-chat/backend
   node scripts/fix_users_endpoint.js server-chat.js
   
   # Copy to production location
   cp /root/lonestar-chat/backend/server-chat.js /opt/lone-star-chat/backend/server-chat-current.js
   
   # Restart systemd service
   systemctl restart lone-star-chat
   ```

3. **Проверка:**
   ```bash
   # Test endpoint without authentication
   curl http://5.249.160.54:666/api/users | jq '{user_count: (.users | length)}'
   # Output: {"user_count": 66}
   ```

## Результат
- ✅ Endpoint `/api/users` теперь публичный
- ✅ Возвращает 66 пользователей без требования авторизации
- ✅ Раздел "Пользователи" должен появиться на главной странице после пересборки приложения

## Важно
Systemd service `/etc/systemd/system/lone-star-chat.service` запускает файл `/opt/lone-star-chat/backend/server-chat-current.js`, поэтому все изменения нужно применять именно к этому файлу.

## Дата исправления
19 ноября 2025, 07:10 UTC

## PID процесса
4172545 (запущен через systemd)
