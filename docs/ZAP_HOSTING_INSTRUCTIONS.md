# 🎯 ФИНАЛЬНАЯ ИНСТРУКЦИЯ - Настройка Backend

**Сервер:** 5.249.160.54  
**Хостинг:** ZAP-Hosting GmbH (Германия)  
**Проблема:** SSH порт 22 закрыт  
**Решение:** Использовать ZAP-Hosting веб-панель

---

## 🖥️ Доступ к ZAP-Hosting панели:

### Вариант 1: Через ZAP-Hosting Dashboard

1. **Войти в личный кабинет ZAP-Hosting:**
   - URL: https://zap-hosting.com/en/customer/
   - Или: https://zap-hosting.com/de/customer/

2. **Найти ваш сервер:**
   - IP: 5.249.160.54
   - Перейти в управление сервером

3. **Открыть Web Console (веб-терминал):**
   - В панели управления найти "Console" или "Terminal"
   - Откроется веб-терминал с root доступом

---

## 🔓 Шаг 1: Открыть SSH (в веб-консоли)

```bash
# Проверить статус SSH
systemctl status sshd

# Запустить SSH
systemctl start sshd
systemctl enable sshd

# Открыть порт в firewall
ufw allow 22/tcp
ufw reload

# Или через iptables
iptables -I INPUT -p tcp --dport 22 -j ACCEPT
iptables-save

# Проверить что порт открыт
ss -tlnp | grep :22
```

---

## 📝 Шаг 2: Обновить backend код

### Способ A: Через веб-консоль (рекомендуется)

```bash
# 1. Перейти в директорию
cd /root

# 2. Создать бэкап
cp server-chat.js server-chat.js.backup-$(date +%Y%m%d-%H%M%S)

# 3. Создать патч файл
cat > category-patch.js << 'EOF'
const fs = require('fs');

// Читаем файл
let code = fs.readFileSync('server-chat.js', 'utf8');

// Находим POST /api/channels endpoint
const searchPattern = /const newChannel = \{([^}]+)\}/s;
const match = code.match(searchPattern);

if (match) {
  const oldCode = match[0];
  
  // Проверяем, нет ли уже category
  if (!oldCode.includes('category:')) {
    // Добавляем category после type
    const newCode = oldCode.replace(
      /type,/,
      `type,
    category: req.body.category || 'chats',`
    );
    
    code = code.replace(oldCode, newCode);
    fs.writeFileSync('server-chat.js', code);
    console.log('✅ Category field added successfully!');
  } else {
    console.log('⚠️  Category field already exists');
  }
} else {
  console.log('❌ Could not find newChannel object');
}
EOF

# 4. Запустить патч
node category-patch.js

# 5. Проверить изменения
grep -A 10 "const newChannel = {" server-chat.js | grep category

# 6. Перезапустить сервер
pm2 restart server-chat

# 7. Проверить что сервер работает
pm2 status
```

### Способ B: Через загрузку скрипта (если SSH открылся)

```bash
# На локальном компьютере:
cd "/Users/svetanaborvinskaia/Desktop/Lone Star Chat"
scp add_missing_endpoints.js root@5.249.160.54:/root/

# На сервере:
ssh root@5.249.160.54
node add_missing_endpoints.js
pm2 restart server-chat
```

### Способ C: Ручное редактирование

Если веб-консоль недоступна, использовать File Manager:

1. В ZAP-Hosting панели открыть File Manager
2. Найти `/root/server-chat.js`
3. Найти строку (~500-600):
   ```javascript
   const newChannel = {
     id: generateId(),
     name,
     description: description || '',
     type,
   ```

4. Добавить после `type,`:
   ```javascript
     category: req.body.category || 'chats',
   ```

5. Сохранить файл
6. В веб-консоли: `pm2 restart server-chat`

---

## ✅ Шаг 3: Проверка работы

### На сервере:

```bash
# Проверить логи
pm2 logs server-chat --lines 20

# Проверить что сервер работает
curl http://localhost/api/channels -I

# Проверить структуру канала
cat /root/data/channels.json | grep -A 5 "category"
```

### С локального компьютера:

```bash
# Проверить API
curl http://5.249.160.54/api/health

# Проверить что backend отвечает
curl http://5.249.160.54/api/channels \
  -H "Content-Type: application/json" \
  2>&1 | grep -o "error\|Authentication"
```

---

## 🎯 Быстрая диагностика

```bash
# Скопировать и вставить в веб-консоль:
echo "=== Backend Status ===" && \
pm2 status && \
echo "" && \
echo "=== SSH Status ===" && \
systemctl status sshd | grep Active && \
echo "" && \
echo "=== Firewall ===" && \
ufw status | head -5 && \
echo "" && \
echo "=== Listening Ports ===" && \
ss -tlnp | grep -E "(:22|:80|:443)"
```

---

## 📞 Контакты ZAP-Hosting

**Если нужна помощь:**

- **Website:** https://zap-hosting.com
- **Support:** https://zap-hosting.com/en/customer/support/
- **Email:** support@zap-hosting.com
- **Документация:** https://zap-hosting.com/guides/

**Запрос в поддержку:**
```
Subject: Please open SSH port 22 on server 5.249.160.54

Hello,

I need SSH access to my server:
IP: 5.249.160.54
Port 22 is currently closed/filtered.

Could you please:
1. Open SSH port 22
2. Ensure SSH service is running
3. Configure firewall to allow SSH connections

Thank you!
```

---

## 🎊 После настройки

### Проверить что категории работают:

1. **В приложении:**
   - Создать новый канал
   - Выбрать "Voice" или "Chats"
   - Создать канал
   - ✅ Канал должен появиться в правильной категории

2. **На сервере:**
   ```bash
   # Посмотреть созданные каналы
   cat /root/data/channels.json | jq '.channels[] | {name: .name, category: .category}'
   ```

3. **Результат:**
   ```json
   {
     "name": "general",
     "category": "chats"
   }
   {
     "name": "voice-chat",
     "category": "voice"
   }
   ```

---

## 📊 Чеклист выполнения:

- [ ] Войти в ZAP-Hosting панель
- [ ] Открыть веб-консоль
- [ ] Проверить статус SSH: `systemctl status sshd`
- [ ] Открыть порт 22: `ufw allow 22/tcp`
- [ ] Создать бэкап: `cp server-chat.js server-chat.js.backup`
- [ ] Добавить поле category (патч или вручную)
- [ ] Перезапустить backend: `pm2 restart server-chat`
- [ ] Проверить логи: `pm2 logs server-chat`
- [ ] Протестировать в приложении

---

## ⏱️ Примерное время:

- **С веб-консолью:** 10-15 минут
- **Через File Manager:** 20-25 минут
- **Через поддержку:** 30-60 минут

---

**Дата:** 10 декабря 2025  
**Статус:** ⏳ Ожидает действий  
**Приоритет:** Низкий (приложение работает без этого)

**Успехов! 🚀**
