# ⚠️ ПРОБЛЕМА С SSH ДОСТУПОМ

## Сервер 5.249.160.54 - SSH порт 22 закрыт

К сожалению, я не могу напрямую подключиться к серверу через SSH, так как порт 22 закрыт или SSH работает на другом порту.

---

## 🔧 ЧТО НУЖНО СДЕЛАТЬ ВРУЧНУЮ

Вам нужно выполнить эти шаги **вручную на сервере**:

### Шаг 1: Подключитесь к серверу через ваш обычный терминал/PuTTY

```bash
ssh root@5.249.160.54
# Пароль: 54654250pS3123
```

**Если SSH на другом порту**, используйте:
```bash
ssh -p НОМЕР_ПОРТА root@5.249.160.54
```

---

### Шаг 2: Создайте файл скрипта на сервере

Скопируйте этот код и создайте файл:

```bash
cat > /root/add_missing_endpoints.js << 'ENDOFFILE'
#!/usr/bin/env node
const fs = require('fs');
const path = require('path');

if (process.argv.length < 3) {
  console.error('❌ Использование: node add_missing_endpoints.js /path/to/server-chat.js');
  process.exit(1);
}

const serverFile = process.argv[2];
if (!fs.existsSync(serverFile)) {
  console.error(\`❌ Файл не найден: \${serverFile}\`);
  process.exit(1);
}

console.log(\`📄 Читаем файл: \${serverFile}\`);
let content = fs.readFileSync(serverFile, 'utf8');

const backupFile = \`\${serverFile}.backup.\${Date.now()}\`;
fs.writeFileSync(backupFile, content);
console.log(\`💾 Создана резервная копия: \${backupFile}\`);

const endpoints = \`

// ==================== ДОБАВЛЕННЫЕ ЭНДПОИНТЫ 9 ДЕКАБРЯ 2025 ====================

// 1. Обновление ролей работника
app.put('/api/admin/employees/:id/roles', authenticateToken, isMaster, (req, res) => {
  const employeeId = parseInt(req.params.id);
  const { roles } = req.body;

  if (!roles || !Array.isArray(roles)) {
    return res.status(400).json({ message: 'Roles must be an array' });
  }

  const users = getUsersFromFile();
  const userIndex = users.users.findIndex(u => u.id === employeeId);

  if (userIndex === -1) {
    return res.status(404).json({ message: 'Employee not found' });
  }

  users.users[userIndex].roles = roles;
  users.users[userIndex].role = roles[0] || 'user';
  saveUsersToFile(users);

  console.log(\\\`✅ Updated roles for user \\\${employeeId}: \\\${roles.join(', ')}\\\`);
  res.json({ message: 'Roles updated successfully', user: users.users[userIndex] });
});

// 2. Удаление работника
app.delete('/api/admin/employees/:id', authenticateToken, isMaster, (req, res) => {
  const employeeId = parseInt(req.params.id);

  if (employeeId === 1) {
    return res.status(403).json({ message: 'Cannot delete master account' });
  }

  const users = getUsersFromFile();
  const userIndex = users.users.findIndex(u => u.id === employeeId);

  if (userIndex === -1) {
    return res.status(404).json({ message: 'Employee not found' });
  }

  const deletedUser = users.users[userIndex];
  users.users.splice(userIndex, 1);
  saveUsersToFile(users);

  console.log(\\\`🗑️  Deleted user \\\${employeeId}: \\\${deletedUser.full_name}\\\`);
  res.json({ message: 'Employee deleted successfully', deleted_user: deletedUser });
});

// 3. Создание нового работника
app.post('/api/admin/employees', authenticateToken, isMaster, async (req, res) => {
  const { email, full_name, password, roles } = req.body;

  if (!email || !full_name || !password) {
    return res.status(400).json({ message: 'Email, full_name and password are required' });
  }

  const users = getUsersFromFile();
  const emailExists = users.users.some(u => u.email.toLowerCase() === email.toLowerCase());

  if (emailExists) {
    return res.status(400).json({ message: 'Email already exists' });
  }

  const maxId = Math.max(...users.users.map(u => u.id), 0);
  const newId = maxId + 1;

  const bcrypt = require('bcrypt');
  const hashedPassword = await bcrypt.hash(password, 10);

  const newUser = {
    id: newId,
    email: email,
    full_name: full_name,
    password: hashedPassword,
    roles: roles || ['user'],
    role: (roles && roles[0]) || 'user',
    avatar_url: null,
    created_at: new Date().toISOString()
  };

  users.users.push(newUser);
  saveUsersToFile(users);

  const { password: _, ...userWithoutPassword } = newUser;
  console.log(\\\`✨ Created new user \\\${newId}: \\\${full_name} (\\\${email})\\\`);
  res.status(201).json({ message: 'Employee created successfully', user: userWithoutPassword });
});

// 4. Получить DM сообщения
app.get('/api/messages/dm/:userId/messages', authenticateToken, (req, res) => {
  const otherUserId = parseInt(req.params.userId);
  const currentUserId = req.user.id;

  const messages = getMessagesFromFile();
  const dmMessages = messages.messages.filter(msg => {
    if (msg.channel_type !== 'dm') return false;
    return (msg.sender_id === currentUserId && msg.recipient_id === otherUserId) ||
           (msg.sender_id === otherUserId && msg.recipient_id === currentUserId);
  });

  dmMessages.sort((a, b) => new Date(a.created_at) - new Date(b.created_at));
  console.log(\\\`📩 Fetched \\\${dmMessages.length} DM messages between users \\\${currentUserId} and \\\${otherUserId}\\\`);
  res.json({ messages: dmMessages });
});

// 5. Отправить DM сообщение
app.post('/api/messages/dm/:userId/messages', authenticateToken, (req, res) => {
  const recipientId = parseInt(req.params.userId);
  const senderId = req.user.id;
  const { content } = req.body;

  if (!content || content.trim() === '') {
    return res.status(400).json({ message: 'Message content is required' });
  }

  const users = getUsersFromFile();
  const recipient = users.users.find(u => u.id === recipientId);

  if (!recipient) {
    return res.status(404).json({ message: 'Recipient not found' });
  }

  const messages = getMessagesFromFile();
  const maxId = Math.max(...messages.messages.map(m => m.id), 0);
  const newId = maxId + 1;

  const newMessage = {
    id: newId,
    sender_id: senderId,
    recipient_id: recipientId,
    channel_id: null,
    channel_type: 'dm',
    content: content.trim(),
    created_at: new Date().toISOString(),
    reactions: {}
  };

  messages.messages.push(newMessage);
  saveMessagesToFile(messages);

  console.log(\\\`💬 New DM message \\\${newId} from user \\\${senderId} to user \\\${recipientId}\\\`);
  res.status(201).json({ message: 'Message sent successfully', data: newMessage });
});

function getMessagesFromFile() {
  try {
    const messagesPath = path.join(__dirname, 'database', 'messages.json');
    if (!fs.existsSync(messagesPath)) return { messages: [] };
    return JSON.parse(fs.readFileSync(messagesPath, 'utf8'));
  } catch (error) {
    console.error('Error reading messages:', error);
    return { messages: [] };
  }
}

function saveMessagesToFile(data) {
  try {
    const messagesPath = path.join(__dirname, 'database', 'messages.json');
    fs.writeFileSync(messagesPath, JSON.stringify(data, null, 2));
  } catch (error) {
    console.error('Error saving messages:', error);
  }
}

// ==================== КОНЕЦ ДОБАВЛЕННЫХ ЭНДПОИНТОВ ====================

\`;

const listenIndex = content.lastIndexOf('app.listen');
let insertIndex = listenIndex !== -1 ? listenIndex : content.length;

const newContent = content.slice(0, insertIndex) + endpoints + content.slice(insertIndex);
fs.writeFileSync(serverFile, newContent);

console.log(\`✅ Эндпоинты успешно добавлены в \${serverFile}\`);
console.log(\`\\n📋 Добавлено 5 эндпоинтов:\`);
console.log(\`   1. PUT /api/admin/employees/:id/roles\`);
console.log(\`   2. DELETE /api/admin/employees/:id\`);
console.log(\`   3. POST /api/admin/employees\`);
console.log(\`   4. GET /api/messages/dm/:userId/messages\`);
console.log(\`   5. POST /api/messages/dm/:userId/messages\`);
console.log(\`\\n🔄 Теперь перезапустите сервер:\`);
console.log(\`   systemctl restart lone-star-chat\`);
ENDOFFILE
```

---

### Шаг 3: Найдите файл сервера

```bash
find /opt /srv /root -name "*server*.js" -type f 2>/dev/null
```

**Обычные места:**
- `/opt/lone-star-chat/backend/server-chat.js`
- `/opt/lone-star-chat/backend/server-chat-current.js`
- `/srv/Lone Star Chat/backend/server.js`

---

### Шаг 4: Запустите скрипт

```bash
# Замените путь на тот что нашли в шаге 3
node /root/add_missing_endpoints.js /opt/lone-star-chat/backend/server-chat.js
```

---

### Шаг 5: Перезапустите сервер

```bash
systemctl restart lone-star-chat
```

**ИЛИ**

```bash
pm2 restart lone-star-chat
```

---

### Шаг 6: Проверьте что работает

```bash
curl http://localhost:80/api/version/check
```

Должен вернуть: `{"version":"...","status":"ok"}`

---

## ✅ ГОТОВО!

После выполнения этих шагов все 7 проблем будут решены:
- ✅ Эмодзи скрыты (код)
- ✅ Текст переведен (код)
- ✅ Категория исправлена (код)
- ✅ API ролей добавлен (сервер)
- ✅ API удаления добавлен (сервер)
- ✅ API создания добавлен (сервер)
- ✅ DM чаты добавлены (сервер)

**Откройте мобильное приложение и протестируйте!** 🎉
