# Импорт реальных сотрудников Mercedes-Benz Lone Star Calgary

**Дата**: 18 ноября 2025  
**Статус**: ✅ Phase 1 завершена

## 🎯 Цель

Заменить 14 тестовых пользователей на реальных сотрудников Mercedes-Benz Lone Star Calgary с сайта:
https://www.mercedes-benz-lonestarcalgary.ca/about-us/staff/

## ✅ Что выполнено - Phase 1

### 1. Импортировано 25 реальных сотрудников из Sales отдела

**Структура данных**:
- ✅ Полное имя (Full Name)
- ✅ Должность (Position)
- ✅ Email (firstname_lastname@lonestar.local)
- ✅ Username (firstname_lastname)
- ✅ Password (firstname_lastname546)
- ✅ Роли (Sales, Master, Lot Team)
- ✅ Аватарки (загружены с сайта)
- ✅ Телефон
- ✅ Department

### 2. Все 25 аватарок успешно загружены

Фотографии сохранены в: `backend/uploads/avatars/`

Примеры:
- `avatar-wayne_desrosiers.png`
- `avatar-simon_clarke.jpg`
- `avatar-rui_gao.png`
- `avatar-grant_yooun.jpg`
- И другие...

### 3. Удалены тестовые аккаунты

**Удалено**: 14 тестовых пользователей  
**Сохранено**: Master Administrator (admin/admin123)

### 4. Система назначения ролей

Роли назначаются автоматически на основе должности:

| Должность | Роль в системе |
|-----------|----------------|
| General Manager | `master`, `sales` |
| Sales Manager, Sales Consultant | `sales` |
| Pre-Owned Sales Consultant | `sales` |
| Business Office Manager | `sales` |
| Delivery Specialist | `sales` |
| Vans Manager | `sales` |
| Site Coordinator / Lot Attendant | `lot_team` |

## 👥 Список добавленных сотрудников (25 из Sales)

### Management (2)
1. **Wayne Desrosiers** - General Manager ⭐ (Master + Sales)
2. **Simon Clarke** - General Sales Manager

### Sales Managers (4)
3. **Grant Yooun** - Sales Manager
4. **Jordan Purcell** - Inventory Manager / National Dealer Trade Contact
5. **John Sweeney** - Deputy Manager & Pre-Owned
6. **Mitch Amos** - Pre-Owned Manager

### Business Office (3)
7. **Cathi Philibert** - Business Office Manager
8. **Connor Marshall** - Business Office Manager
9. **Tracy Garcia** - Business Office Manager

### Sales Consultants (10)
10. **Chris Pedulla** - Sales Consultant
11. **Dardana Stavileci** - Sales Consultant
12. **Himi Sharma** - Sales Consultant
13. **John Faul** - Sales Consultant
14. **Kaylee Coosemans** - Sales Consultant
15. **Kevin Fong** - Sales Consultant
16. **Nicholl Cilic** - Sales Consultant
17. **Tyson Lee** - Sales Consultant
18. **Miles Woolley** - Sales Consultant
19. **Brandon Corbett** - Delivery Specialist

### Pre-Owned & Vans (4)
20. **Shef Hirani** - Vans Manager
21. **Ali Khosravi** - Pre-Owned Sales Consultant
22. **Ralph Leith** - Pre-Owned Sales Consultant
23. **Rui Gao** - Pre-Owned Sales Consultant

### Operations (2)
24. **Valentin Ursan** - Used Car Manager Assistant
25. **Jean Nadeau** - Site Coordinator / Lot Attendant (Lot Team role)

## 🔐 Формат учетных данных

### Пример: Rui Gao
```
Username: rui_gao
Email: rui_gao@lonestar.local
Password: rui_gao546
Role: Sales
Avatar: /uploads/avatars/avatar-rui_gao.png
```

### Формула генерации
```
Username: <firstname>_<lastname> (lowercase, spaces → underscore)
Email: <username>@lonestar.local
Password: <username>546
```

### Master Administrator (сохранен)
```
Username: admin
Email: admin@lonestar.local
Password: admin123
Role: Master
```

## 📊 Статистика

| Метрика | Значение |
|---------|----------|
| **Всего пользователей в системе** | 26 |
| - Master Administrator | 1 |
| - Реальные сотрудники Sales | 25 |
| **Загружено аватарок** | 25 |
| **Размер базы данных** | users.json |
| **Тестовых аккаунтов удалено** | 14 |

## 📁 Структура файлов

```
backend/
├── database/
│   └── users.json ← Обновлен (26 пользователей)
├── uploads/
│   └── avatars/
│       ├── avatar-1-1762593772076.jpg (Master)
│       ├── avatar-wayne_desrosiers.png
│       ├── avatar-simon_clarke.jpg
│       ├── avatar-rui_gao.png
│       └── ... (еще 22 аватарки)
└── scripts/
    └── import_real_employees.js ← Скрипт импорта
```

## 🚀 Как запустить импорт

```bash
cd backend
node scripts/import_real_employees.js
```

## 🔄 Phase 2 (Будущее)

Для добавления оставшихся 40 сотрудников из других отделов:

### Service Department (26 сотрудников)
- Service Manager
- Service Advisors
- Technicians
- Parts Integration

### Parts Department (9 сотрудников)
- Parts Manager
- Parts Advisors
- Parts Counter Staff

### Admin Department (5 сотрудников)
- Accounting
- HR
- IT Support
- Reception
- Office Manager

**Итого Phase 2**: 40 сотрудников  
**Общий итог после Phase 2**: 65 сотрудников + 1 Master = 66 пользователей

## 📝 Инструкция для Phase 2

1. Откройте сайт: https://www.mercedes-benz-lonestarcalgary.ca/about-us/staff/
2. Переключитесь на вкладки: Service, Parts, Admin
3. Скопируйте данные каждого сотрудника:
   - Полное имя
   - Должность
   - Email
   - Фото URL
4. Добавьте данные в `scripts/import_real_employees.js` в соответствующие секции
5. Запустите скрипт снова

## ✅ Проверка работы

### 1. Проверить количество пользователей
```bash
cat database/users.json | grep '"full_name"' | wc -l
# Должно быть: 26
```

### 2. Проверить аватарки
```bash
ls -1 uploads/avatars/ | wc -l
# Должно быть: 26+ файлов
```

### 3. Тестовый вход
```
Username: rui_gao
Password: rui_gao546
```

Или любой другой сотрудник из списка!

## 🎉 Результат

✅ **Phase 1 успешно завершена!**

- 25 реальных сотрудников из Sales добавлены
- Все аватарки загружены
- Тестовые аккаунты удалены
- Master Admin сохранен
- Система готова к тестированию

---

**Следующий шаг**: Протестировать вход в приложение с новыми реальными аккаунтами!
