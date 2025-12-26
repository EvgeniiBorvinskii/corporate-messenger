# ✅ AI Chat - Управление через Rules

**Дата:** 20 ноября 2025  
**Версия:** Alpha 0.31  
**Сервер:** 5.249.160.54:666 (PID 1030402)  
**Сборка:** 77.6MB, 56.4s

---

## 🎯 Проблема

**До исправления:**
- AI Chat был **выключен** по умолчанию в настройках (`show_ai_chat: false`)
- Но кнопка AI Chat **всегда отображалась** на экране
- Master не мог управлять отображением AI кнопки
- Настройка в Rules не влияла на UI

**Запрос:**
> "AI Chat показывает что выключен, но он включен по факту. Давай сделаем что бы AI чат по стандарту был включен при установке приложения, но мастер аккаунт может его выключить для всех пользователей через вкладку Rules"

---

## ✅ Решение

### 1. Mobile App - Загрузка Rules

**Файл:** `mobile/lib/screens/home/home_screen.dart`

**Добавлено:**
```dart
// Rules state
final ApiService _apiService = ApiService();
Map<String, dynamic> _rules = {
  'show_ai_chat': true, // По умолчанию включен
};
bool _rulesLoaded = false;

// В initState()
_loadRules();

Future<void> _loadRules() async {
  try {
    final response = await _apiService.get('/api/admin/rules');
    if (response.containsKey('rules')) {
      setState(() {
        _rules = Map<String, dynamic>.from(response['rules']);
        _rulesLoaded = true;
      });
      print('✅ Rules loaded: show_ai_chat = ${_rules['show_ai_chat']}');
    }
  } catch (e) {
    print('⚠️ Error loading rules: $e');
    // При ошибке используем дефолтные значения (AI включен)
    setState(() {
      _rulesLoaded = true;
    });
  }
}
```

---

### 2. Условная отрисовка AI кнопки

**Было:**
```dart
// Draggable AI Button with Physics
Positioned(
  left: screenSize.width / 2 + _aiButtonPosition.dx - 30,
  top: screenSize.height * 0.7 + _aiButtonPosition.dy,
  child: GestureDetector(
    onTap: () {
      if (!_isDragging) {
        context.go('/ai-chat');
      }
    },
    child: _buildAIButton(),
  ),
),
```

**Стало:**
```dart
// Draggable AI Button with Physics (только если включен в настройках)
if (_rulesLoaded && (_rules['show_ai_chat'] == true))
  Positioned(
    left: screenSize.width / 2 + _aiButtonPosition.dx - 30,
    top: screenSize.height * 0.7 + _aiButtonPosition.dy,
    child: GestureDetector(
      onTap: () {
        if (!_isDragging) {
          context.go('/ai-chat');
        }
      },
      child: _buildAIButton(),
    ),
  ),
```

**Ключевые изменения:**
- Обернули `Positioned` в `if (_rulesLoaded && (_rules['show_ai_chat'] == true))`
- AI кнопка отображается **только если** rules загружены **И** флаг включен
- При ошибке загрузки rules используется дефолтное значение `true`

---

### 3. Дефолтное значение в Admin Panel

**Файл:** `mobile/lib/screens/admin/admin_panel_screen.dart`

**Было:**
```dart
Map<String, dynamic> _rules = {
  'show_schedule_button': true,
  'show_ai_chat': false,  // ❌ Выключен
  'show_users_section': false,
};
```

**Стало:**
```dart
Map<String, dynamic> _rules = {
  'show_schedule_button': true,
  'show_ai_chat': true,  // ✅ Включен по умолчанию
  'show_users_section': false,
};
```

---

### 4. Backend - Rules Endpoints

**Файл:** `backend/server-chat-current.js`

**Добавлено через скрипт `add_rules_endpoints.js`:**

```javascript
// Глобальные настройки приложения
let appRules = {
  show_schedule_button: true,
  show_ai_chat: true,  // ✅ По умолчанию включен
  show_users_section: false,
};

// GET /api/admin/rules - Получить настройки
app.get('/api/admin/rules', authenticate, async (req, res) => {
  try {
    console.log('📋 Getting rules:', appRules);
    res.json({ rules: appRules });
  } catch (error) {
    console.error('❌ Error getting rules:', error);
    res.status(500).json({ error: 'Failed to get rules' });
  }
});

// PUT /api/admin/rules - Обновить настройки (только для Master)
app.put('/api/admin/rules', authenticate, async (req, res) => {
  try {
    const userId = req.user?.id;
    const user = users[userId];
    
    // Проверка прав Master
    if (!user || !user.roles || !user.roles.includes('master')) {
      return res.status(403).json({ 
        error: 'Only Master can update rules',
        required_role: 'master'
      });
    }

    const updates = req.body;
    console.log('📝 Updating rules:', updates);
    
    // Обновляем настройки
    appRules = { ...appRules, ...updates };
    
    console.log('✅ Rules updated:', appRules);
    res.json({ 
      success: true,
      rules: appRules 
    });
  } catch (error) {
    console.error('❌ Error updating rules:', error);
    res.status(500).json({ error: 'Failed to update rules' });
  }
});
```

**Особенности:**
- Глобальная переменная `appRules` хранит настройки
- GET endpoint возвращает текущие rules
- PUT endpoint обновляет rules (только Master)
- Проверка роли `master` перед обновлением
- Дефолтное значение: `show_ai_chat: true`

---

## 🔄 Deployment

### Backend

```bash
# 1. Загрузка скрипта
scp backend/scripts/add_rules_endpoints.js root@5.249.160.54:/root/

# 2. Выполнение
ssh root@5.249.160.54 "node /root/add_rules_endpoints.js /opt/lone-star-chat/backend/server-chat-current.js"

# 3. Перезапуск
ssh root@5.249.160.54 "sudo systemctl restart lone-star-chat"
```

**Результат:**
```
✅ Rules endpoints успешно добавлены!
   GET /api/admin/rules - получить настройки
   PUT /api/admin/rules - обновить (только Master)
   
⚙️ Дефолтные значения:
   show_ai_chat: true  ✅ Включен по умолчанию

● lone-star-chat.service - Active: active (running)
  Main PID: 1030402 (node)
  🚀 Lone Star Chat API running on port 666
```

### Mobile App

```bash
cd "/Users/svetanaborvinskaia/Desktop/Lone Star Chat/mobile"
flutter build ios --release
```

**Результат:**
```
✓ Built build/ios/iphoneos/Runner.app (77.6MB)
Время: 56.4 секунды
```

---

## 🎯 Как это работает

### При запуске приложения

1. **Загрузка настроек:**
   ```
   App загружается
     ↓
   home_screen.dart → initState()
     ↓
   _loadRules() вызывает GET /api/admin/rules
     ↓
   Backend возвращает: { rules: { show_ai_chat: true } }
     ↓
   setState: _rules = {...}, _rulesLoaded = true
   ```

2. **Отрисовка UI:**
   ```
   build() вызывается
     ↓
   Проверка: if (_rulesLoaded && _rules['show_ai_chat'] == true)
     ↓
   Если TRUE: показываем AI кнопку
   Если FALSE: не показываем AI кнопку
   ```

### Изменение настроек Master'ом

1. **Master открывает Admin Panel → Rules**
2. **Выключает AI Chat** (переключатель становится OFF)
3. **Нажимает Save**
4. **PUT /api/admin/rules** отправляется на сервер:
   ```json
   { "show_ai_chat": false }
   ```
5. **Backend обновляет** `appRules.show_ai_chat = false`
6. **Все пользователи при следующей загрузке** не увидят AI кнопку

---

## 📊 Состояния AI Chat

| Ситуация | show_ai_chat | AI кнопка отображается? |
|----------|--------------|-------------------------|
| Первая установка | `true` (дефолт) | ✅ ДА |
| Master выключил | `false` | ❌ НЕТ |
| Master включил обратно | `true` | ✅ ДА |
| Ошибка загрузки rules | `true` (fallback) | ✅ ДА |
| Нет интернета | `true` (fallback) | ✅ ДА |

**Важно:** При ошибке загрузки rules используется дефолтное значение `true`, чтобы AI Chat оставался доступным.

---

## 🔐 Безопасность

### Проверка прав Master

```javascript
const user = users[userId];

if (!user || !user.roles || !user.roles.includes('master')) {
  return res.status(403).json({ 
    error: 'Only Master can update rules',
    required_role: 'master'
  });
}
```

**Только Master** может изменять настройки через PUT /api/admin/rules.

**Обычные пользователи:**
- Могут получать rules через GET /api/admin/rules
- **НЕ могут** изменять через PUT (получат 403 Forbidden)

---

## ✅ Проверка работы

### 1. AI Chat включен (дефолт)

**При первой установке:**
```
✅ Пользователь видит AI кнопку на Home экране
✅ Может перетаскивать кнопку
✅ Клик открывает AI Chat экран
```

**В Admin Panel → Rules:**
```
✅ Переключатель "AI Chat" в позиции ON
✅ Зеленая галочка рядом с названием
```

### 2. Master выключает AI Chat

**Действия:**
1. Master открывает Admin Panel
2. Переходит на вкладку Rules
3. Выключает переключатель "AI Chat" (OFF)
4. Нажимает Save

**Результат:**
```
✅ Сервер получает: { show_ai_chat: false }
✅ Backend обновляет: appRules.show_ai_chat = false
✅ Успешный ответ: { success: true, rules: {...} }
✅ UI обновляется: переключатель OFF, красный крестик
```

### 3. Все пользователи перезапускают приложение

**Что происходит:**
```
App загружается
  ↓
GET /api/admin/rules → { rules: { show_ai_chat: false } }
  ↓
_rules['show_ai_chat'] = false
  ↓
Проверка: if (_rulesLoaded && _rules['show_ai_chat'] == true) → FALSE
  ↓
❌ AI кнопка НЕ отображается
```

### 4. Master включает обратно

**Действия:**
1. Master открывает Rules
2. Включает переключатель "AI Chat" (ON)
3. Сохраняет

**Результат:**
```
✅ Backend: appRules.show_ai_chat = true
✅ Все пользователи (после перезапуска) снова видят AI кнопку
```

---

## 📝 Изменённые файлы

### Mobile (Frontend)

1. **mobile/lib/screens/home/home_screen.dart**
   - Добавлена загрузка rules через `_loadRules()`
   - Добавлено состояние: `_rules`, `_rulesLoaded`
   - AI кнопка обернута в `if (_rulesLoaded && _rules['show_ai_chat'] == true)`
   - Fallback на `true` при ошибке загрузки

2. **mobile/lib/screens/admin/admin_panel_screen.dart**
   - Изменено дефолтное значение: `'show_ai_chat': true`

### Backend

1. **backend/server-chat-current.js**
   - Добавлена переменная `appRules` с дефолтными значениями
   - Добавлен `GET /api/admin/rules` endpoint
   - Добавлен `PUT /api/admin/rules` endpoint (только Master)
   - Проверка роли `master` перед обновлением

2. **backend/scripts/add_rules_endpoints.js** (новый)
   - Автоматическое добавление rules endpoints в сервер
   - Создает backup перед изменением
   - ~130 строк кода

---

## 🚀 Что работает

### Функционал

✅ AI Chat **включен по умолчанию** при первой установке  
✅ AI кнопка отображается на Home экране (если включено)  
✅ Master может **выключить** AI Chat для всех через Rules  
✅ Master может **включить** обратно через Rules  
✅ Изменения применяются после перезапуска приложения  
✅ При ошибке загрузки rules используется дефолт (включено)  
✅ Проверка прав: только Master может изменять rules  
✅ GET /api/admin/rules доступен всем (с аутентификацией)  
✅ PUT /api/admin/rules доступен только Master  

### Backend

✅ Rules endpoints работают (GET/PUT)  
✅ Глобальная переменная `appRules` хранит настройки  
✅ Проверка роли `master` работает  
✅ Дефолтное значение: `show_ai_chat: true`  
✅ Сервер запущен: PID 1030402 на порту 666  

### Mobile App

✅ Загрузка rules при старте приложения  
✅ Условная отрисовка AI кнопки  
✅ Fallback на дефолтные значения  
✅ iOS сборка успешна (77.6MB, 56.4s)  
✅ Admin Panel отображает правильный статус  

---

## 🎉 Итоги

**Задача:** Сделать AI Chat включенным по умолчанию, но с возможностью выключения Master'ом

**Выполнено:**
- ✅ AI Chat включен по умолчанию (`show_ai_chat: true`)
- ✅ Master может управлять через Admin Panel → Rules
- ✅ AI кнопка отображается только если флаг включен
- ✅ Backend endpoints добавлены (GET/PUT)
- ✅ Проверка прав Master реализована
- ✅ Production deployment завершен
- ✅ iOS app собран (77.6MB)

**Теперь система работает правильно:**
1. 🟢 По умолчанию: AI Chat **включен** для всех
2. 🔴 Master может **выключить** для всех через Rules
3. 🟢 Master может **включить** обратно
4. ✅ Изменения применяются после перезапуска

**Автор:** GitHub Copilot  
**Дата:** 20 ноября 2025, 22:45 UTC
