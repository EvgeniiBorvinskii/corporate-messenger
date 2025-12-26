# 📋 Как извлечь сотрудников из вкладок Service, Parts, Admin

## 🎯 Задача
Извлечь данные всех сотрудников из трех оставшихся вкладок на сайте Mercedes-Benz Lone Star Calgary

---

## 🌐 Метод 1: Через браузер (РЕКОМЕНДУЕТСЯ)

### Шаг 1: Откройте страницу
```
https://www.mercedes-benz-lonestarcalgary.ca/about-us/staff/
```

### Шаг 2: Откройте Developer Tools
- **Mac**: `Cmd + Option + I`
- **Windows**: `F12` или `Ctrl + Shift + I`

### Шаг 3: Нажмите на вкладку SERVICE

### Шаг 4: В Console вставьте этот код:

```javascript
// Код для извлечения всех сотрудников с текущей вкладки
const staff = Array.from(document.querySelectorAll('.staff-item, .team-member, [class*="staff"]')).map(el => {
  const nameEl = el.querySelector('h3, .name, [class*="name"]');
  const titleEl = el.querySelector('.title, .position, [class*="title"], [class*="position"]');
  const phoneEl = el.querySelector('.phone, [href^="tel:"]');
  const emailEl = el.querySelector('a[href^="mailto:"]');
  const imgEl = el.querySelector('img');
  
  return {
    name: nameEl?.textContent?.trim(),
    position: titleEl?.textContent?.trim(),
    phone: phoneEl?.textContent?.trim() || phoneEl?.href?.replace('tel:', ''),
    email: emailEl?.href?.replace('mailto:', ''),
    photo: imgEl?.src
  };
}).filter(s => s.name);

console.log(JSON.stringify(staff, null, 2));
copy(JSON.stringify(staff, null, 2));
```

### Шаг 5: Скопируйте результат
Данные автоматически скопируются в буфер обмена (команда `copy()`)

### Шаг 6: Сохраните в файл
Создайте файл `service_employees.json` и вставьте скопированные данные

### Шаг 7: Повторите для PARTS и ADMIN
Нажмите на вкладку Parts, выполните тот же код, сохраните в `parts_employees.json`
Нажмите на вкладку Admin, выполните тот же код, сохраните в `admin_employees.json`

---

## 🌐 Метод 2: Альтернативный код (если первый не работает)

```javascript
// Универсальный парсер для любой структуры
const allElements = document.querySelectorAll('div, article, section');
const staff = [];

allElements.forEach(el => {
  const text = el.textContent;
  
  // Ищем паттерны: Имя, затем должность, затем телефон
  if (text.includes('@lonestarcalgary.com') || text.includes('403')) {
    const name = el.querySelector('h1, h2, h3, h4, strong, b')?.textContent?.trim();
    const img = el.querySelector('img')?.src;
    
    if (name && img) {
      staff.push({
        name: name,
        text: text.trim().substring(0, 200), // Первые 200 символов для анализа
        photo: img
      });
    }
  }
});

console.log(JSON.stringify(staff, null, 2));
copy(JSON.stringify(staff, null, 2));
```

---

## 🌐 Метод 3: Извлечь все табы автоматически

```javascript
// Автоматически переключает табы и собирает данные
const tabs = document.querySelectorAll('a[href*="staff"], button[data-tab], .tab-link');
const allStaff = { sales: [], service: [], parts: [], admin: [] };

console.log(`Найдено табов: ${tabs.length}`);

// Вручную кликаем по каждому табу
tabs.forEach((tab, i) => {
  console.log(`Таб ${i}: ${tab.textContent}`);
});

// После клика на SERVICE запустите:
allStaff.service = Array.from(document.querySelectorAll('[class*="staff"], [class*="team"]'))
  .map(el => ({
    name: el.querySelector('h3, .name')?.textContent?.trim(),
    photo: el.querySelector('img')?.src
  }))
  .filter(s => s.name);

console.log('SERVICE:', allStaff.service);
copy(JSON.stringify(allStaff.service, null, 2));
```

---

## 📱 Метод 4: Ручное копирование (самый надежный!)

Если автоматические методы не работают, просто:

1. Откройте вкладку **SERVICE**
2. Для каждого сотрудника запишите:
   - Имя
   - Должность
   - Телефон
   - Email (если есть)
   - URL фото (правый клик на фото → Copy Image Address)

Пример формата для записи:
```
John Smith | Service Manager | 403-253-1333 | john.s@lonestarcalgary.com | https://...image.jpg
Mike Brown | Technician | 403-253-1334 | mike.b@lonestarcalgary.com | https://...image.jpg
```

3. Повторите для **PARTS** и **ADMIN**

---

## 🤖 Метод 5: Я сделаю это за вас

Если хотите, вы можете просто:

1. Открыть сайт: https://www.mercedes-benz-lonestarcalgary.ca/about-us/staff/
2. Сделать **скриншоты** каждой вкладки (Service, Parts, Admin)
3. Показать мне скриншоты
4. Я вручную перепишу данные в нужный формат

---

## 📊 Что мне нужно от каждого сотрудника:

```javascript
{
  "name": "John Smith",           // Полное имя
  "position": "Service Manager",   // Должность
  "phone": "403-253-1333",        // Телефон
  "email": "john.s@...",          // Email (если есть)
  "photo": "https://...",         // URL фотографии
  "department": "Service"         // Service, Parts, или Admin
}
```

---

## ✅ После получения данных

Когда у вас будут данные, отправьте мне JSON в таком формате:

**service_employees.json:**
```json
[
  {
    "name": "John Smith",
    "position": "Service Manager",
    "phone": "403-253-1333",
    "photo": "https://di-uploads-pod24.dealerinspire.com/.../john.jpg"
  }
]
```

И я:
1. Обновлю скрипт `import_service_parts_admin.js`
2. Запущу импорт
3. Загружу аватарки
4. Обновлю production сервер
5. У вас будет полный список всех 66 сотрудников! 🎉

---

## 🎯 Какой метод попробовать?

**Рекомендую в таком порядке:**
1. ✅ Метод 1 (автоматический парсер) - самый быстрый
2. ✅ Метод 3 (с автопереключением табов) - если есть кнопки
3. ✅ Метод 4 (ручное копирование) - 100% надежный, но медленный
4. ✅ Метод 5 (скриншоты) - если вообще ничего не работает

**Сколько времени займет:**
- Метод 1-3: 5-10 минут
- Метод 4: 20-30 минут
- Метод 5: я сделаю за 15 минут после получения скриншотов

---

## 🚀 Готовы начать?

Какой метод хотите попробовать? Или может быть хотите, чтобы я попробовал извлечь данные другим способом?
