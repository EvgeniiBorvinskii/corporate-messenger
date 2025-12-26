# 🌟 Lone Star Chat

# 🌟 Lone Star Chat

**Полностью автономная, приватная, локальная цифровая платформа для автодилершипа**

[![Production Ready](https://img.shields.io/badge/status-production%20ready-green)]()
[![Flutter](https://img.shields.io/badge/Flutter-3.0+-blue)]()
[![Node.js](https://img.shields.io/badge/Node.js-20%20LTS-green)]()
[![TypeScript](https://img.shields.io/badge/TypeScript-5.1-blue)]()
[![Docker](https://img.shields.io/badge/Docker-Compose-blue)]()
[![LLaMA 3](https://img.shields.io/badge/LLaMA-3%208B-orange)]()
[![License](https://img.shields.io/badge/license-proprietary-red)]()

---

## 🎯 Что это?

Lone Star Chat — это **корпоративная мессенджер-платформа нового поколения** для автодилершипа с численностью персонала ~100 человек. Полностью автономная, приватная, локальная — без облаков, без внешних API, без компромиссов в безопасности.

### ✨ Ключевые возможности

| Функция | Описание |
|---------|----------|
| 🔒 **100% Privacy** | Все данные на вашем сервере. Никаких облачных сервисов. |
| 🤖 **Local AI** | LLaMA 3 8B с RAG для ответов на вопросы о компании |
| 💬 **Real-time Chat** | Личные, групповые чаты, файлы, голосовые сообщения |
| 📞 **Voice Calls** | WebRTC аудиозвонки без внешних серверов |
| 📅 **Calendar** | Планирование встреч, напоминания, синхронизация |
| 🎨 **Liquid Glass** | Современный дизайн вдохновленный Apple |
| 🔄 **Force Update** | Контроль версий с принудительным обновлением |
| 🚀 **Auto Deploy** | Установка одной командой, автоматические бэкапы |

---

## 🏗️ Архитектура

## 🎯 Обзор

Lone Star Chat — это локальная платформа для ~100 сотрудников с AI-ассистентом, работающая на вашем собственном сервере без облачных сервисов, внешних API или зависимостей от третьих сторон.

### Ключевые характеристики:
- 🔒 **100% приватность** — все данные остаются на вашем сервере
- 🤖 **Локальный AI** — LLaMA 3 8B (GGUF 4-bit) через llama.cpp
- 💻 **Без GPU** — оптимизировано для CPU (Ryzen 9 / Xeon)
- 🎨 **Liquid Glass дизайн** — премиум UI в стиле Apple
- 📱 **Flutter мобильное приложение** — iOS и Android
- 🛡️ **Безопасность** — UFW, Fail2Ban, изоляция Docker
- 📊 **RAG архитектура** — устойчивое обучение AI на корпоративных данных

## 🏗️ Архитектура

```
┌─────────────────────────────────────────────────────────────┐
│                    МОБИЛЬНЫЕ КЛИЕНТЫ                        │
│              (iOS & Android - Flutter)                      │
│        + Splash Screen (Lone Star Chat.mp4)                 │
└────────────────────┬────────────────────────────────────────┘
                     │ HTTPS/WSS
                     │
┌────────────────────▼────────────────────────────────────────┐
│                       NGINX (Reverse Proxy)                 │
│                   SSL/TLS Termination                       │
└────────┬───────────────────────────┬────────────────────────┘
         │                           │
         │                           │
┌────────▼──────────┐      ┌────────▼──────────────────────┐
│   ADMIN PANEL     │      │      API SERVER               │
│  (Flutter Web)    │      │    (Node.js/Express)          │
│                   │      │                               │
│ • Пользователи    │      │ • REST API                    │
│ • Версии          │      │ • WebSocket (чаты)            │
│ • force_update    │      │ • JWT Auth                    │
│ • Мониторинг      │      │ • Version Control             │
│ • RAG контент     │      │ • WebRTC (голос)              │
└───────────────────┘      └───────┬───────────────────────┘
                                   │
                   ┌───────────────┼───────────────┐
                   │               │               │
         ┌─────────▼────┐  ┌──────▼──────┐  ┌────▼─────────┐
         │ PostgreSQL   │  │   Redis     │  │ Meilisearch  │
         │              │  │             │  │              │
         │ • Users      │  │ • Sessions  │  │ • RAG Index  │
         │ • Messages   │  │ • Cache     │  │ • Vectors    │
         │ • Schedules  │  │ • PubSub    │  │ • Documents  │
         │ • Settings   │  └─────────────┘  └──────────────┘
         └──────────────┘
                   │
         ┌─────────▼──────────────┐
         │    llama.cpp Server    │
         │   (LLaMA 3 8B GGUF)    │
         │                        │
         │ • CPU Inference        │
         │ • 4-bit Quantization   │
         │ • Context: 4096        │
         │ • RAG Integration      │
         └────────────────────────┘
```

## 📦 Структура проекта

```
/srv/Lone Star Chat/
├── backend/                    # API сервер
│   ├── src/
│   │   ├── controllers/       # REST endpoints
│   │   ├── models/            # DB models
│   │   ├── services/          # Business logic
│   │   ├── middleware/        # Auth, validation
│   │   ├── rag/               # RAG pipeline
│   │   └── websocket/         # WebSocket handlers
│   ├── Dockerfile
│   ├── package.json
│   └── tsconfig.json
├── mobile/                     # Flutter мобильное приложение
│   ├── lib/
│   │   ├── screens/           # UI screens
│   │   ├── widgets/           # Reusable components
│   │   ├── services/          # API client
│   │   ├── models/            # Data models
│   │   └── theme/             # Liquid Glass theme
│   ├── assets/
│   │   └── videos/
│   │       └── Lone Star Chat.mp4
│   ├── pubspec.yaml
│   └── ios/android/
├── admin/                      # Flutter Web админка
│   ├── lib/
│   │   ├── pages/             # Admin pages
│   │   ├── widgets/           # Admin components
│   │   └── services/          # Admin API
│   └── pubspec.yaml
├── models/                     # AI модели
│   ├── llama-3-8b-instruct-q4_k_m.gguf
│   └── README.md
├── scripts/                    # Скрипты автоматизации
│   ├── install.sh             # Установка системы
│   ├── backup.sh              # Резервное копирование
│   ├── update.sh              # Обновление компонентов
│   ├── monitor.sh             # Мониторинг
│   └── restore.sh             # Восстановление
├── docs/                       # Документация
│   ├── DEPLOYMENT.md          # Развертывание
│   ├── ARCHITECTURE.md        # Архитектура
│   ├── TRAINING.md            # Обучение персонала
│   ├── SCALING.md             # Масштабирование
│   ├── LEGAL.md               # Юридические аспекты
│   └── API.md                 # API документация
├── docker-compose.yml         # Оркестрация контейнеров
├── .env.example               # Пример переменных окружения
└── README.md                  # Этот файл
```

## 🚀 Быстрый старт

### Предварительные требования

- Ubuntu Server 22.04
- 64-128 GB RAM
- Ryzen 9 / Xeon процессор
- Docker и Docker Compose
- 200+ GB свободного места

### Установка

```bash
# 1. Клонировать или скопировать проект в /srv/Lone Star Chat
cd "/srv/Lone Star Chat"

# 2. Запустить скрипт установки
sudo bash scripts/install.sh

# 3. Настроить переменные окружения
cp .env.example .env
nano .env

# 4. Загрузить LLaMA 3 модель
bash scripts/download-model.sh

# 5. Запустить систему
docker-compose up -d

# 6. Проверить статус
docker-compose ps
bash scripts/monitor.sh
```

### Первый запуск

1. Откройте админку: `https://your-server-ip/admin`
2. Войдите с дефолтными credentials (измените сразу!)
3. Создайте пользователей
4. Настройте версию приложения и force_update
5. Загрузите корпоративные документы для RAG

## 📱 Сборка мобильного приложения

### iOS

```bash
cd mobile
flutter pub get
flutter build ios --release
# Открыть в Xcode для подписи и публикации
open ios/Runner.xcworkspace
```

### Android

```bash
cd mobile
flutter pub get
flutter build appbundle --release
# APK находится в build/app/outputs/bundle/release/
```

## 🔧 Конфигурация

### Основные параметры (.env)

```env
# Сервер
NODE_ENV=production
PORT=3000
HOST=0.0.0.0

# База данных
POSTGRES_HOST=postgres
POSTGRES_PORT=5432
POSTGRES_DB=lonestar
POSTGRES_USER=lonestar
POSTGRES_PASSWORD=CHANGE_ME

# Redis
REDIS_HOST=redis
REDIS_PORT=6379

# Meilisearch
MEILI_HOST=http://meilisearch:7700
MEILI_MASTER_KEY=CHANGE_ME

# JWT
JWT_SECRET=CHANGE_ME_STRONG_SECRET
JWT_EXPIRES_IN=7d

# LLaMA
LLAMA_MODEL_PATH=/models/llama-3-8b-instruct-q4_k_m.gguf
LLAMA_CONTEXT_SIZE=4096
LLAMA_THREADS=8

# Приложение
APP_VERSION=1.0.0
FORCE_UPDATE=false
```

## 🛡️ Безопасность

- **Firewall**: UFW настроен на порты 80, 443, 22
- **Fail2Ban**: Защита от брутфорса
- **SSL**: Let's Encrypt сертификаты
- **JWT**: Token-based аутентификация
- **Docker изоляция**: Внутренняя сеть для сервисов
- **Rate limiting**: API защита от перегрузки
- **Логирование**: Все действия записываются

## 📊 Мониторинг

```bash
# Проверить здоровье сервисов
bash scripts/monitor.sh

# Просмотр логов
docker-compose logs -f api
docker-compose logs -f llama

# Статистика ресурсов
docker stats
```

## 💾 Резервное копирование

```bash
# Создать backup
bash scripts/backup.sh

# Восстановить из backup
bash scripts/restore.sh /path/to/backup.tar.gz
```

## 🔄 Обновление

```bash
# Обновить компоненты
bash scripts/update.sh

# Обновить AI модель
bash scripts/update-model.sh
```

## 📖 Документация

Подробная документация находится в папке `/docs`:

- [Архитектура](docs/ARCHITECTURE.md)
- [Развертывание](docs/DEPLOYMENT.md)
- [API](docs/API.md)
- [Обучение персонала](docs/TRAINING.md)
- [Масштабирование](docs/SCALING.md)
- [Юридические аспекты](docs/LEGAL.md)

## 🆘 Поддержка

### Частые проблемы

**Проблема**: llama.cpp не запускается
```bash
# Проверить логи
docker-compose logs llama
# Проверить модель
ls -lh models/
```

**Проблема**: Медленные ответы AI
```bash
# Увеличить количество потоков
# В .env: LLAMA_THREADS=16
docker-compose restart llama
```

**Проблема**: Ошибки подключения к БД
```bash
# Проверить PostgreSQL
docker-compose exec postgres psql -U lonestar -d lonestar -c "SELECT 1;"
```

## 📈 Производительность

### Рекомендуемые настройки для 100 пользователей:

- **CPU**: 16+ cores (Ryzen 9 5950X / Xeon E5)
- **RAM**: 64 GB (рекомендуется 128 GB)
- **Storage**: NVMe SSD 500GB+
- **LLaMA потоки**: 8-16
- **PostgreSQL connections**: 100
- **Redis maxmemory**: 8GB

### Ожидаемая производительность:

- Время ответа API: <100ms
- Время генерации AI (короткий текст): 2-5 секунд
- Время генерации AI (длинный текст): 10-30 секунд
- Одновременные пользователи: 100+
- Сообщения в секунду: 1000+

## 🔮 Будущие улучшения

- [ ] Интеграция с внешними API (опционально)
- [ ] Голосовые сообщения
- [ ] Видеозвонки
- [ ] Push уведомления
- [ ] Машинное обучение на локальных данных
- [ ] Мультиязычная поддержка
- [ ] Темная тема
- [ ] Desktop приложение (Electron)

## 📄 Лицензия

Проприетарное ПО. Все права защищены.

## 👥 Команда

Разработано для автодилершипа с заботой о приватности и безопасности.

---

**Версия**: 1.0.0  
**Дата**: 2025-10-04  
**Статус**: Production Ready 🚀
